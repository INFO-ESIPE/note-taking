[[Network Services]]
****
**Table of Contents**
```table-of-contents
```

****
## Domain Name

### Why?

As explained in the previous lecture, an IP address identifies a machine on the internet (globally).
There can be of two types:
- **IPv4**: 32-bit notation, represented as four decimal integers ranging from 0 to 255, separated by periods `193.50.159.151`
- **IPv6**: 128-bit notation, represented in hexadecimal format, typically consisting of eight groups of four hexadecimal digits separated by colons.
  `2620:0:862:ed1a:bc42:c0:10a0:1`

The problem is clear: these addresses are not easy to remember...
This is where DNS comes into play: A distributed service that allows, among other features, the binding of a domain name to an IP address.

> [!info]- Before DNS
> Initially, A `HOSTS.TXT` file (RFC 608 & 1974) was attributing hostnames by organisations in their internal network, without any global structure.
> Quickly introduced several issues:
> - Bottlenecks on Stanford's NIC network
> - Slow updates to the `HOSTS.TXT` file
> - The `HOSTS.TXT` file was becoming excessively large

### Domains

A **domain** is a grouping of machines that fall under the same **administrative authority**.
	*Note that domains are unrelated to geographical localisation of machines*

![[domain.png|500]]
> [!example]
> `fr`, `com`, `u-pem.fr`, `gouv.fr`... are domains

Domains are organised in a hierarchical tree structure, with the root being the "`.`" domain. Each node or leaf in this tree represents a domain. The name of a domain (label) can be up to 63 bytes long.
> [!example]
> In the domain `berkeley.edu`, `berkeley` is a child label of the `edu` label.

To access a specific host, we write it, followed by all the domains. For example, if we want to access `www.google.com`: 
	Host = www
	Domain 2 = google
	Domain 1 = com

### The DNS Service

A DNS server publishes all characteristics of the domains it has authority over, including:
- Names and IP addresses of hosts
- Existing services (Mail, LDAP, etc.)
- Email of the administrator
- ...
These characteristics are defined in **Ressource Records (RR)**.

![[dns-nutshell.png|400]]

### DNS Zone

A **DNS Zone** refers to a domain and its subdomains that are **managed under the same administrative authority**. 
For instance, `.fr` and `.asso.fr` belong to the same zone as they are both managed by the same authority (l'AFNIC).

The DNS Protocol is used to obtain information about a domain through the exchange of requests and responses between the client and the DNS server. This includes:
    IPv4 address of a host
    Address of the domain's DNS server
    Name of the domain's email servers...

### Domains Space

Domains are organised like a tree:

![[dns-tree.jpg|700]]

The root domain is managed by IANA.


Domains such as `com`, `uk`, `org`... are known as **Top level domains (TLD)**. IANA delegates its authority on TLDs to entities called **Registry Operators**.
> [!example] Examples of Registry Operators
> `.com` and`.net` are managed by VeriSign; `.fr` is managed by AFNIC...
> Registry operators must publish all information about the TLDs they manage on a public **WHOIS** server.  

TLDs can be of two types:
- **Country Code TLD (ccTLD)**: Associated to a country
	*`.fr`, `.uk`, `?ca`, `.jp`...*
- **Generic TLD (gTLD)**: Initially used by american organisations, they now have very diverse meanings
	*`.net`, `.edu`, `.com`, `.mango`, `.yoga`...*
*TLDs are listed here: www.iana.org/domains/root/db*


Domains placed below TLDs are known as **Second-Level Domains (SLD)**. They are delegated by the Registry Operator of the TLD to entities (e.g., companies, universities) that manage and operate the domain.
	`elysee.fr is` an SLD of the `.fr` TLD.


A **Fully Qualified Domain Name (FQDN)** shows all the labels constituting it.
	*`ns.v6.berkeley.edu`, `www.u-pem.fr`...*
A Non-FQDN is called a **Partially Qualified Domain Name (PQDN)**.
> [!info]
> Convention dictates that when editing a DNS configuration file, FQDNs should be followed by a dot to signify the root domain.
>	*`ns.v6.berkeley.edu.`
>	`www.u-pem.fr.`*


***
## Resolve

There are thirteen root servers, and each knows the IP Address of all TLDs' DNS servers.
To avoid bottlenecks, those servers have replicas all around the world.x

> [!info]- Root servers around the world
> They all belong to the `root-servers.net` domain:
> 
> | **Root Server**    | **Organisation / Country**              |
> | ------------------ | --------------------------------------- |
> | a.root-servers.net | VeriSign / USA                          |
> | b.root-servers.net | University of Southern California / USA |
> | c.root-servers.net | Cogent Communications / USA             |
> | d.root-servers.net | University of Maryland / USA            |
> | e.root-servers.net | NASA / USA                              |
> | f.root-servers.net | Internet Systems Consortium / USA       |
> | g.root-servers.net | US Department of Defence / USA          |
> | h.root-servers.net | US Army Research Lab / USA              |
> | i.root-servers.net | Netnod / Sweden                         |
> | j.root-servers.net | VeriSign / USA                          |
> | k.root-servers.net | RIPE NCC / Netherlands                  |
> | l.root-servers.net | ICANN / USA                             |
> | m.root-servers.net | WIDE Project / Japan                    |
> 
> We can localise them (and their replicas) here: https://root-servers.org/
> ![[root-servers.png]]

A server which has authority on a DNS zone is known as an **Authoritative Name Server**.
It shall know (at least):
- All Resource Record of its zone
- IP Addresses of sub-zones' Name Server (NS)
- IP Addresses of root NS

> [!example]-
> ![[nameserver.png|400]]
> 1. Here, root NS knows the address of `.fr`'s DNS.
> 2. `.fr`'s NS knows the IP address and hostname of: 
> - All root DNS
> - All DNS of `wikipedia.fr` and `u-pem.fr`, and all RR of the `.fr` domain
> 3. `u-pem.fr`'s NS knows the IP address and hostname of:
> - All root DNS
> - `www` and `ent` hosts, and all RR of the `u-pem.fr` domain

In general, there are at least two DNS servers per zone:
- **Primary server (mandatory)**: Configured by the domain administrator, it contains the official RR list of the zone
- **Secondary server(s)**: Copies of the primary server, mostly here in case the main server is down or too solicited
A DNS server can also store data in cache (Response to requests, additional RRs provided by a NS it queried).
The cache duration is defined by a **Time To Live (TTL)** value given by the queried NS before answer.

### Client-side

DNS is an application-layer protocol using port 53 (for both UDP and TCP).
The client machine must know the IP address of at least one DNS Server (in general, its the ISP/Organisation's NS).
	*Defined in `/etc/resolv.conf` on Linux.*

The **resolver** is usually provided by the operating system and manages name resolution for applications.
It can cache resolved RRs to reduce query latency. On Linux, caching requires enabling tools like **nscd** or **systemd-resolved**.

![[resolver.png|500]]
> [!info]
> It can conserve discovered RR in cache. 
> This is often done on Windows, but more rarely on Linux; services like "nscd" or "systemd-resolved" must be enabled.

So, if an application requires a DNS record:
- It sends a request to the client resolver
- The resolver transmits the request to one of the NS he knows
	- If the NS knows the RR (has authority on the zone or cached RR), it answers to the client
	- If it doesn't, the NS must manually query another DNS servers. It is called **recursive**.

![[resolver-example.png|500]]

> [!question] What if the DNS we recursively query has empty cache?
> - The recursive server must forward the query to a root DNS server, which will forward him the IP address of the TLD's DNS server (DNS Server of `.fr` in the example above).
> - The recursive server now must transmit the request to this newly acquired DNS server, which will forward the IP address of the SLD's DNS server (DNS Server of `u-pem.fr` in the example above).
> - The recursive server now queries this final DNS where the domain belongs, and obtain the RR. It forwards it to the client's resolver.

### Server-side

A Recursive DNS server can store previously acquired RRs in cache.
	*For instance, if a recursive NS queried a root NS for `.com`, it will cache `.com`'s NS. Next time it is asked, it won't ask to ask a root NS again as the information is already acquired and stored in cache.*

Both root and TLD servers are always **iterative**:
- They dont't forward a DNS request to another server
- They always return the address of a lower-layer NS.

On the other hand, SLD servers or lower **can be iterative or recursive**.
	*The recursive mode is optional. In general, a NS is only recursive for machines that belongs in his zone.*

> [!example]- Example: Resolve (SLD in Iterative mode)
> Hypothesis: all cache is empty
> ![[dns-iterative.png|700]]

> [!example]- Example: Cache usage
> ![[dns-cache.png|700]]


***
## DNS Frames

### Overall frame

Composed of five parts

![[dns-frame.jpg]]
1. **Header**: Informations about the message
	*Message type (answer or request), message Id, indicates client preferences for resolution (recursive or iterative) and what the DNS server supports*
2. **Question**: Defines the RR concerned by the message
3. **Answer**: Contains the RR concerned
4. **Authority**: Contains the RRs pointing to the answer's authoritative DNS 
5. **Additional**: Can carry further information (unasked RR...)

### Header

If we focus on the header, here's what it looks like:
 - **ID**: Message id (same for question and answer)
> - **QR**: 0 for a request, 1 for an answer
> - **OPCODE**: Type of question
> 	- 0 = Standard (direct resolution)
> 	- 1 = Reverse resolution
> 	- 2 = Get server status
> - **AA (Authoritative Answer)**: 1 if the server has authority, 0 otherwise
> - **TC (TrunCation)**: 1 if the message was truncated
> - **RD (Recursion Desired)**: 1 is put in the request if the client wishes for recursivity. The 1 remains in the answer if the server supports recursivity.
> - **RA (Resource Available)**: 1 if the server supports recursivity
> - **RCODE (Response Code)**: Indicates if an error occurred in the answer
> 	*0 = no error, 1 = format error, 2 = server error, 3 = hostname non-existant, 4 = unsupported request, 5 = server refuses to answer*
> - **QDCOUNT, ANCOUNT, NSCOUNT, ARCOUNT**: Number of entries in their [[02 - DNS#Other sections|respective section]]

### Other sections

**Question**:
- **QNAME**: A Domain
- **QTYPE**: The type (A, PTR, ...)
- **QCLASS**: The class (IN, CH)


**RR Format in other sections**
- **NAME**: A Domain
- **TYPE**: The type (A, PTR, ...)
- **CLASS**: The class (IN, CH)
- **TTL**: TIme to live
- **RDLENGTH**
- **RDATA**

### At transport layer

Requests/Answers are encapsulated in either:
- UDP if the message size is within the standard limit (512 bytes for traditional DNS or up to 4096 bytes with [[02 - DNS#DNS Extensions|EDNS]]).
- TCP if the message exceeds the size limit, or for reliability and security reasons (e.g., [[02 - DNS#DNS Extensions|DNS over TLS]] or zone transfers).

Nowadays, DNS answers are very large so its common for them to use TCP.
	*Mostly due to deployment of IPv6 (long IPs), usage of DNSSEC, and usage of non-latin characters requiring large encoding*

> [!question] What exactly happens if the message is too long?
> The server fills the **TC** bit in the header, and forwards the answer.
> Then, the client must send its request again, but via TCP.
> 


***
## DNS Records

### Concept

Each DNS record has the following components:
1. **Owner**: The domain or host associated with the record.
2. **Type**: Specifies the resource the record refers to (e.g., IPv4 address, alias, or mail server).
3. **Class**: Defines the protocol family (commonly `IN` for Internet).
4. **TTL (Time to Live)**: Specifies how long the record can be cached.
5. **RDATA**: The resource-specific data.

> [!example]
> `www.google.com. IN A 192.168.1.1`
> Owner: www.google.com
> Class: IN
> Type: A
> RDATA: 192.168.1.1

### Common Types

| **Type** | **Description**                                                                                     | **Example**                                                                                    |
| -------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `A`      | Maps a hostname to an IPv4 address.                                                                 | `example.com. IN A 93.184.216.34`                                                              |
| `AAAA`   | Maps a hostname to an IPv6 address.                                                                 | `example.com. IN AAAA 2606:2800:220:1:248:1893:25c8:1946`                                      |
| `NS`     | Specifies authoritative DNS servers for a domain.                                                   | `example.com. IN NS ns1.example.com.`                                                          |
| `SOA`    | Provides domain ownership and operational details (e.g., serial, refresh, retry intervals).         | `example.com. IN SOA ns1.example.com. admin.example.com. (2024010101 3600 1800 1209600 86400)` |
| `MX`     | Defines mail servers for the domain, sorted by priority.                                            | `example.com. IN MX 10 mail1.example.com.`                                                     |
| `CNAME`  | Creates an alias for a domain.                                                                      | `mail.example.com. IN CNAME smtp.example.com.`                                                 |
| `PTR`    | Used for reverse DNS lookups, mapping an IP to a hostname.                                          | `34.216.184.93.in-addr.arpa. IN PTR example.com.`                                              |
| `TXT`    | Stores text data (e.g., SPF records, domain verification).                                          | `example.com. IN TXT "v=spf1 include:_spf.google.com ~all"`                                    |
| `SRV`    | Specifies services offered by the domain, including transport protocol, port, priority, and weight. | `_sip._tcp.example.com. IN SRV 10 20 5060 sipserver.example.com.`                              |

### Master file

A DNS server's database is composed of three structures:
- **Master files**, each one textually describing the RRs of the zone it is linked to
- Configuration files declaring all zones and pairing them to their master file
- Cache

> [!example]- DNS Server Structure
> `/etc/bind/named.conf.local` file declaring zones:
> ```bash
> zone "esipe.mlv"{                      # 1
> 	type master ;
> 	file "/etc/bind/db.esipe.mlv" ;
> }
> 
> zone "44.55.193.in-addr-arpa"{         # 2
> 	type master ;
> 	file "/etc/bind/db.44.55.193" ;
> }
> 
> zone "0.5.0.0.2.b.a.9.1.0.2.ip6.arpa"{ # 3
> 	type master ;
> 	file "/etc/bind/db.ip6.arpa" ;
> }
> ``` 
> 
> Master file for each zone:
> ```bash
> # 1
> $ORIGIN esipe.mlv.
> $TTL 3600
> @ IN SOA dns1.esipe.mlv.  root.esipe.mlv. {
> 	201911271 ;
> 	604800 ;
> 	# ...
> }
> 
> # 2
> $ORIGIN 44.55.193.in-addr-arpa.
> $TTL 3600
> @ IN SOA dns1.esipe.mlv.  root.esipe.mlv. {
> 	201911271 ;
> 	604800 ;
> 	# ...
> }
> 
> # 3
> $ORIGIN 0.5.0.0.2.b.a.9.1.0.2.ip6.arpa.
> $TTL 3600
> @ IN SOA dns1.esipe.mlv.  root.esipe.mlv. {
> 	201911271 ;
> 	604800 ;
> 	# ...
> }
> ```
> 
> Structure for cache:
> ```bash
> www.ietf.org. 3600 IN A 104.20.1.85
> u-pem.fr. 1800 IN NS helios.u-pem.fr.
> # ...
> ```
> 
> Usage of the "@" character indicates the origin zone.
>	*It avoids having to write the complete FQDN each time.
>	As "@" is used for this purpose, it can't be used to define emails anymore !! In this context, it must be replaced by a dot*
>
> `$ORIGIN` represents the domain to which all PQDN RRs belongs in the current file.

The bare minimum for a master file is to contain the SOA of the zone. Each line defines an RR, following this format:
`name   class   type   RData`


***
## Reverse Lookup

In short, reverse DNS maps IP addresses back to domain names.
	*We obtain the PTR registry from an IP, instead of retrieving the A registry from a domain name*

Two dedicated domains were created for this purpose:
1. `in-addr.arpa` for IPv4
2. `ip6.arpa` for IPv6 (`ip6.int` in the past)
Reverse DNS servers for these domains are managed hierarchically. Regional Internet Registries (RIRs) manage address blocks for continents and delegate control over smaller ranges to Internet Service Providers (ISPs) or organisations.


***
## DNS Extensions

EDNS enhances DNS by allowing a server to specify its supported maximum message size for UDP
This is specified in a pseudo-RR known as OPT (OPTion, type 41). This RR contains the size (which can be up to 4096 bytes), but can also contain flags indicating support for other extensions (e.g., DNSSEC)  

> [!info]- ECS: EDNS for CDNs
> A protocol known as EDNS Client Subnet (ECS) was designed specifically for content delivery networks.
> It allows a recursive DNS to bind the client's network prefix to the request.
>	*The ultimate goal of this is to localise the client via its IP address, so the DNS can return the IP of a replica that is near their location.*

DNSSEC adds a signature to answers, ensuring
	- To authentify the answer (ensure it was emitted by the server which has authority on the domain)
	- To ensure it was not altered in transit
*Note that DNSSEC does not guarantee confidentiality, only integrity and authentication!*

DNS over TLS (DoT) crypts both requests and answers. It is supported by most DNS servers nowadays...
