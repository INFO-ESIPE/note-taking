[[Network Security]]
***
**Table of Contents**
```table-of-contents
```

****
## Principle

As its name implies, the purpose is to control an user's access to a network or a service on the network.
	*Notions of authentication and protocols (e.g., Kerberos) are described [[11 - User Authentication|in this lecture]]*

As defined in RFC 2865, the **Network Access Server (NAS)**, also called Remote Access Server (RAS), is the entry point of the network. It collects authentication information.
The **AAA (Authentication, Authorisation, Accounting/Auditing)** server is the one performing the authentication procedure based on what's provided by the NAS.

![[basic-auth.png|675]]

| **NAS**                                                                                                                                         | **AAA**                                                                                                                                |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Supports an authentication protocol (e.g.: Radius, Diameter) and<br>authentication methods (e.g., EAP)                                          | Contains the database allowing authentication, or can access <br>an external authentication system<br>                                 |
| Collects authentication information provided by the user and<br>forwards them to the AAA server                                                 | Executes the same authentication protocol as the NAS contacting it<br>Forwards the authentication result back to the NAS once computed |
| Filters (or block) in/out traffic depending on the authentication<br>success or not<br>(Intervenes in IP routing process, FW and QoS functions) | Can be shared among several NAS (mostly for an ISP)                                                                                    |

So, as described above, both servers communicate using a common authentication protocol (which can remain unknown from the client's machine attempting to authenticate).
However, It is possible for an AAA server to only serve as a proxy for another AAA server, in order to centralise operations.

![[auth-proxy.png|650]]
> [!info]
> This is used for the eduroam service in European universities, or for roaming between two operators


***
## RADIUS
*Remote Authentication Dial-In User Service; RFC 2865*

Transport authentication, authorisation and configuration information between the NAS and AAA following a client-server model (NAS being the client and AAA the server).
It encapsulates four kinds of packets in UDP using port 1812: `Access-Request`, `Access-Accept`, `Access-Reject`, `Access-Challenge`

The principle is: There is a **shared secret** between client (NAS) and server (AAA) to authenticate their exchanges (usually, a signature or HMAC).
Both entities supports several methods of authentication. Any information they exchange is encrypted.

### Session

User sends authentification parameters to the NAS (login/password or a certificate, along with the resource asked).
NAS sends an `Access-Request` packet containing:
- User parameters (encrypted via the shared secret with AAA)
- Port and NAS which received the user request

The AAA can now:
1. Authentify the NAS
2. Check user credentials by using the database (or forward to another server if RADIUS proxy)
3. Sends back a response to the NAS: **Success**, **failure** or **challenge**

![[radius-session.png|680]]

The `Access-Reject` packet will be sent if authentication failed.
On the other hand, an `Access-Accept` packet will be sent if authentication is successful. It may carry information needed by the end-user to use the service or resource he tries to access.
	*IP address, QoS parameters...*

Finally, the `Access-Challenge` can be sent by the server. Its a classical challenge/response mechanism.
AAA server transmits a binary sequence, which the NAS forwards to the user. The user must send back an encrypted version of this sequence.
	*Several challenges can occur in RADIUS session, and several protocols can be used for this purpose (CHAP, MD5, OTP...)*

> [!example]- Authentication on eduroam
> As briefly mentioned above, its an authentication system shared among European universities and research centres.
> It allows moving users to authenticate themselves on other organisations' networks and access the Internet.
> 
> Under the hood, it uses a RADIUS server hierarchy:
> - One RADIUS server in each university's network
> - One RADIUS proxy in each country (managed by RENATER in France)
> - One root RADIUS proxy (managed by GEANT)
>   
> If Alice (from EPFL school in Switzerland) authenticates on eduroam from UGE school in France, it will behave like so:
> ![[eduroam.png|700]] 

> [!example]- Integrity check mechanism
> ![[radius-integrity.png|700]]

### Packet

Look like this:
![[radius-format.png|700]]
- **Code** is the type of packet (1 for `Access-Request`, 2 for `Access-Accept`, 3 for `Access-Reject`, 11 for `Access-Challenge`)
- **Identifier** is an identifier given to a request. Response to this request will carry the same identifier, so it can be linked to it
- **Length** indicates the total length of the packet in bytes (between 20 and 4096)
- **Response Authenticator** is the signature of the server
	`MD5 [ (Code + ID + Length + RA) request + attributes + secret) ]`
- **Request Authenticator (RA)** is an unpredictable numerical value chosen by the client 

For an attribute:
- **Type** (1 for `User-Name`, 2 for `User-Password`, 3 for `CHAP-Password`, 4 for `NAS-IP-Address`, 5 for `NAS-Port`, or 79 for `EAP-Message`)


***
## EAP
*Extensible Authentication Protocol; RFC 3748*

EAP is a **framework** which can be used to **implement an authentication protocol**, along with the **transport of authentication information** between user and AAA. 
	*NAS can ignore the authentication method which was chosen*

EAP does not require routing, it can be carried on the data link layer (either IEEE 802.1x -> Switched LAN, IEEE 802.11i -> Wireless LAN).
It is flexible: Authentication mechanism can be *chosen* among the ones mentioned in the RFC (e.g., OTP, MD5-challenge, RSA Public Key Authentication).

![[eap-vocabulary.png|720]]

> [!example]- EAP exchange
> ![[eap-exchange.png|700]]

### Packet

Looks like this:
![[eap-packet.png|600]]
- **Code** indicates the purpose of the packet (1 for `Request`, 2 for `Response`, 3 for `Success`, 4 for `Failure`)
- **Type** is the authentication method's id (defined by IANA)
	*A few examples: 13 = EAP-TLS, 4 = MD5-Challenge, 5 = One Time Password, 9 = RSA Public Key Authentication*

### EAP-TLS
*RFC 5216*

The most widely-used and safe method. It authentifies both the supplicant and the EAP server via X.509 certificates.
	*EAP-TLS only leverages the authentication phase of TLS, but does not establish a full TLS session for encryption*

![[eap-tls.png|550]]