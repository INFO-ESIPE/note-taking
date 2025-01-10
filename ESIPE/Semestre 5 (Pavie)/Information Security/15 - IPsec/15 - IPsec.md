[[Information Security]]
***
**Table of Contents**
```table-of-contents
```

***
## Why?

Applications already implement their own security mechanisms (S/MIME for Email, Kerberos for Active Directory, TLS for web access), but we would like **to have an IP-Level security mechanism, introducing IPsec.**. 
	*While aforementioned mechanism are implemented at the application layer (OSI Level 7) or presentation layer (OSI Level 6), IPsec will operate at the network layer (OSI Level 3).*

![[osi-model.png]]

The point of a lower-level security mechanism is **to provide a generic security mechanism for any kind of application or communication** (while higher-level ones only works for specific applications).

We would like our security protocol to provide:
- **Authentication**: Validates that the data originates from a legitimate source.
- **Confidentiality**: Encrypts data to prevent unauthorised access.
- **Key Management**: Securely exchanges keys for cryptographic operations.


***
## Concept

IPsec ultimately allows to **secure end-user-to-end-user traffic via encryption and authentication at the IP-Level**, thus mitigating risks of unauthorised monitoring and control of network traffic.

But what are the concrete benefits of IPsec, well:
1. **Transparent Security**: Works below the transport layer (TCP/UDP), so it is invisible to applications and end-users.
2. **Perimeter Security**: Can be deployed on firewalls or routers to protect all traffic crossing the network boundary.
3. **Compatibility**: Functions seamlessly with both IPv4 and IPv6, upper-layer software remains unaffected.
4. **Routing Architecture**: Secures essential routing communications:
    - Validates router advertisements and neighbour advertisements.
    - Prevents forged routing updates.

> [!info]
> To this day, we only use IPsec-v2 (RFC 2401, 1998) and IPsec-v3 (RFC 4301, 2005), as the first one is completely obsolete.


***
## Components

It looks like this:

![[ipsec-archi.png]]
> [!note]
> Since we aren't in a networking class (nor an algorithmic class), we will only focus on the two protocols of the upper layer. 

### IPsec modes (not mandatory for class)

IPSec provides two modes of operation to cater to different use cases: one for protecting specific data payloads and another for encapsulating entire packets.
- **Transport Mode**: Protects only the IP payload (SDU). The original IP header remains unchanged. This mode is typically used for host-to-host communication.

- **Tunnel Mode**: Protects the entire IP packet (PDU) by encapsulating it within a new IP header. This mode is commonly used for network-to-network or host-to-network communication, such as in VPNs.


### Authentication Header (AH)

First solution for encapsulation. Provides data integrity, authentication of the IP packet sender, and optional replay protection. 
==However, it does not provide encryption.==
![[ipsec-ah.svg]]


### Encapsulating Security Payload (ESP)

Second solution for encapsulation. Adds both encryption and authentication to ensure confidentiality and integrity.
![[ipsec-esp.svg]]


***
## Commercial VPNs
*Very superficial approach...*

Commercial Virtual Private Networks (VPN) services leverage IPSec or other protocols to establish secure transport-level connections. The **end-user will be connected to a privately-owned exit node**.

![[vpn-exitnode.svg]]

However, keep in mind that commercial VPNs:
- **Cannot hide their own usage**, meaning organisations may block VPN traffic.
- Relies on the exit node's jurisdiction and policies, which may log or monitor traffic.
- Do not protect users from threats originating from the target endpoint.
