[[Information Security]]
11/12/2024
****
**Table of Contents**
```table-of-contents
```

****
## Concept

TLS (current version 1.3, defined by [RFC 8446](https://datatracker.ietf.org/doc/html/rfc8446) ), is an evolution of the deprecated Secure Sockets Layer (SSL) protocol.
Even though we heavily associate it to the web (since decent web browsers embeds the protocol in their features), its available for most application-layer exchanges **over TCP**.
	*TLS is related to the presentation OSI layer. So, it is placed in between the application layer (e.g., HTTP, FTP...) and the session+transport layer (TCP).
	For UDP, the Datagram Transport Layer Security (DTLS) protocol is used.*

It aims to guarantee:
- **Confidentiality**: Ensure that an adversary cannot read the traffic
- **Integrity**: Ensure that an adversary cannot tamper with your traffic
	*Prevent replay attacks from an adversary recording the raw encrypted traffic and then replaying it to the server (replay a packet that sends “Pay $10000 to Mallory”)*
- **Authenticity**: Make sure you’re talking to the legitimate server
	*Defend against an attacker impersonating the server* (pretending to be your bank, for instance)

The RFC specifies that TLS is integrated with several supporting inner protocols that manage various aspects of the exchange:
- **Record Protocol**: The foundational protocol that provides basic services like confidentiality, integrity, and authentication for all TLS communications.
- **Handshake Protocol**: Used to initiate the exchange, establish cryptographic parameters, and collect necessary information (e.g., certificate, premaster secret).
- **Alert Protocol**: Handles error reporting and session termination notifications.
- **Change Cipher Spec Protocol**: Signals a switch to a new set of cryptographic parameters during the handshake.
Additionally, there is a fifth protocol, the **Heartbeat Protocol**, which is defined in its own [RFC](https://datatracker.ietf.org/doc/html/rfc6520)It provides a mechanism to keep the connection alive and check for connectivity.

![[tls.png]]
> The **record protocol** underpins the other protocols and provides essential transport services. This is why it is depicted below the inner protocols in diagrams—it serves as the foundation for all TLS communications.


****
## Inner-Protocols

### Handshake

Obviously, the **two machines (client and server) must already have an active TCP** (fault-tolerant transport layer protocol) **connection with each other**.
The result of this handshake is a **TLS Session** between both parties, allowing the authentication to be "kept in memory" along with **how it was initialised** (cryptographic parameters used for this specific handshake). 
	*Once two parties have created a TLS Session, this session can be used if the same parties prepares new connections (e.g., multithreaded data exchange program will initiate several TCP connections between the same parties, but on different ports)*

Handshake steps:
1. **Hello exchange**
	- Client initiates the handshake with a `ClientHello` message containing a 256-bit random number `Rb`, along with a list of supported cryptographic algorithms he is ready to use.
	- Server **responds** with a `ServerHello` message containing a 256-bit random number `Rs`, along with a cryptographic algorithm he chose from the client's list.
	*`Rs` and `Rb` are here to prevent replay attacks. This phase also includes a few metadatas about the exchange (protocol version, session ID and compression method), but it's not that important*

2. **Certificate**
	- The server sends its certificate (identity and public key, signed by a CA).
	- The client verifies the signature in the certificate
	*Keep in mind that certificates are public. So, the client is still not sure that he isn't talking to an impersonator, as anyone can grab the server's public key.*

3. **Premaster Secret** - This step has two purposes: 
	1. Make sure the **client is talking to the legitimate server** (The server must prove that it owns the private key corresponding to the public key in the certificate)
	2. Make the **client and server share a secret** (To secure the exchange)
	We were able to do this step with either RSA or **Diffie-Hellman Ephemeral (DHE)**, but only DHE is supported since [[12 - TLS#TLS v1.3|v1.3]]. This is how we do it with DHE:
	- The server generates a secret `a`, and then computes `g^a mod p`
	- The server signs this computation with his private key and forwards it to the client along with its signature
	- The client verifies the signature (proves that the server owns the private key), and then generates a secret `b` and send `g^b mod p`
	- The client and server now shares a **premaster secret**: `g^ab mod p` (Like regular Diffie-Hellman, an eavesdropper cannot retrieve `g^ab`).
![[dhe.png]]
> Our handshake looks like this so far

4. **Derive Symmetric Keys** - The server and client each derive symmetric keys from `RB` and `RS` (usually by seeding a [[04 - Stream Ciphers#Randomness|PRNG]] with both values). Four keys are derived:
	- `Cb`: For encrypting client-to-server messages
	- `Cs`: For encrypting server-to-client messages
	- `Ib`: For MACing client-to-server messages
	- `Is`: For MACing server-to-client messages
	*Note that both parties are aware of those four keys*

5. **Exchange MACs** - The server and client exchange MACs over all prior handshake messages using the agreed cryptographic algorithm (from step 1 to 4).

Once this is done, messages can securely be exchanged between the two parties. They will be MACed and then encrypted (for legacy reasons, the other way around is usually better).
![[mac.png]]
> Each message `M` will be encrypted with `Cb`/`Cs`, and MACed with `Ib`/`Is`.


### TLS Record

The foundation of TLS, responsible for encapsulating higher-level protocols (Handshake, Alert, Change Cipher Spec, etc.).
- It handles **fragmentation**, **compression** (if negotiated), and **encryption** of data.
- Ensures confidentiality, integrity, and authentication by applying the agreed cryptographic algorithms.
- Operates independently of the underlying transport protocol, such as TCP.


### Change Cipher

One of the four TLS-specific protocols leveraging the TLS Record protocol.
It's simply a one-byte message that copies the pending state into the current state.


### Alert

Used to notify the peer about **errors** or **session state changes**.
- Alerts are categorised as **warning** (e.g., close notifications) or **fatal** (e.g., handshake failure, bad certificate).
- Each alert consists of:
    1. A **severity level** (warning or fatal).
    2. An **alert description** (e.g., unexpected message, decryption failure).
- A **fatal alert** immediately terminates the session, while warnings allow the session to continue under certain conditions.


### Heartbeat

A heartbeat protocol is usually here to monitor the availability of an entity by sending periodic signals (hence the name of the protocol).
In the case of TLS (and DTLS), this protocol is purely an extension that provides a "keep-alive" functionality.
	*Even if TLS (unlike DTLS) is built over TCP which is a reliable transport protocol, the later does not provide a native keep-alive feature that maintains the connection without continuous data transfer (which is why HTTP provides this keep-alive feature as well).*

The user can use the new `HeartbeatRequest` message, which has to be answered by the peer with a `HeartbeartResponse` immediately. This avoids to drop the connection each time a client is served, and instead keep it working for a long period of time.
	*If the party doesn't answer quickly, we consider that it is disconnected and we proceed into dropping the session. This is mostly a security against session hijacking.*


****
## Attacks

In short, attacks are not against the cipher itself, but the protocol.
	*We usually attack the handshake phase.*

Some common type of attacks: PRNG Sabotage, Unknown Certificate Authority


****
## TLS v1.3

As explained in the [[12 - TLS#Handshake|handshake chapter]], RSA isn't supported anymore.
	*Too risky: If the private key is compromised, all handshakes are compromised. Diffie-Hellman mitigates this risk as new keys are generated for each handshake.*

For performance optimisation, the client sends `g^b mod p` directly in `ClientHello`.