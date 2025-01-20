[[Information Security]]
04/12/2024
****
**Note:** Definitions (especially the ones given in [[11 - User Authentication#Authentication Principles|the first chapter]]) must be known, and with the specific vocabulary used by the teacher (bold text). He is very demanding when it comes to the vocabulary we use, and was very insistent during the lecture, so it might be at the exam...
****
**Table of Contents**
```table-of-contents
```

****
## Authentication Principles

First part of the AAA of cybersecurity: 
- **Authentication**: Ensures you are who you **claim** to be (**identity of users**)
	*This, in fact, extends to **any type of attribute that is recognised by a system** (check age, address, weight...)*
- Authorization: Ensures resource access is based on permissions
- Accountability: Ensures actions from users are tracked

**Identity** is a set of attributes that uniquely identifies an entity among others. 
The **User authentication service** is here to verify the identity claimed by an entity attempting to access the system, in two steps:
1. Identification step - User gives an identifier (typically, username and password) to the security system
2. Verification step - The system try to authenticate the claim the user makes
	*Which means only the user and the authenticating party should be aware of the password*


****
## NIST Authentication Model

The **electronic user authentication** is a widely-used model that establishing confidence in user's identity. 

Registration steps:
1. An applicant applies to a trustworthy **registration authority (RA)** to become a subscriber of a **credential service provider (CSP)**
	*The RA will be in charge of the database containing credentials (typically, email-password couples)*
2. The CSP forwards an **electronic credential** to the subscriber. 
	*The credential is a data structure that authoritatively binds an identity and additional attributes to a token possessed by a subscriber and can be verified when presented to the verifier in an authentication transaction
	This token is either an encrypted password or an encryption key*
3. Token and credential are used for next communications. 

When authenticating, the user that wants to be authenticated (**claimant**) sends his token to the verifying party (**verifier**). If the token is valid, the verifier is now aware than the claimant is the subscriber.


****
## Means of Authentication

Four generic ways of authenticating someone's identity:
- **Something you know**: A password, passphrase, a PIN, answer to a personal question...
- **Something you possess**: Cryptographic key, Electronic keycards ...
	*This is what a token is doing*
- **Something you are** (Any static biometric): Fingerprint, Face, Retina...
- **Something you do** (Any behavioural biometric): Handwriting characteristics, Voice pattern... 


***
## Mutual Authentication

Allows each party to ensure each other's identity, so they can exchange keys safely.
Authenticated key exchange needs to ensure **confidentiality** (with a certificate), and **Timeliness**, to prevent **message replay** attacks.
	*if a session key gets compromised at a later time, we can perform the attack. So, those keys needs to expire (after a very short period of time, if possible).*

### Relay Attacks

To prevent relay attacks, two approaches:
- **Timestamps:** Party A accepts a message as fresh only if this message contains a timestamp that, in A’s judgement, is close enough to A’s knowledge of current time
	*Clocks of both parties must be synchronised, via NTP for instance*
> This approach is not suitable for connection-oriented applications, as we need a way to reliably synchronise the clocks (difficult and prone to errors).

- **Challenge/response:** Party A, expecting a fresh message from B, first sends B a nonce (challenge) and requires that the subsequent message (response) received from B contain the correct nonce value
	*A sends a random number to B. B needs to encrypt it with the public key, and forward the encrypted nonce. If A can decrypt it and retrieve the original nonce, the challenge is validated.*
> This approach is not suitable for connection-less applications, as the challenge behaves like a handshake (which is a connection). 
> It breaks the principle of connection-less as we have to effectively connect before making our connection-less transactions afterwards).


****
04/12/2024
## Kerberos

Kerberos is an authentication protocol based on **symmetric encryption**. It allows a **user** to access resources given by **service providers** securely over a network.

It operates in **distributed client/server infrastructures**, where one or more **Kerberos servers**handle authentication procedures.

To ensure reliability, Kerberos is designed to resist the following types of attacks:
- **impersonation**: Adversary gains access to a victim's machine (or credentials) and pretends to be him
- **Spoofing**: Adversary modifies the IP/MAC address of their machine to masquerade as a legitimate one
- **Hijacking**: Adversary intercepts valid tokens or credentials during communication to replay them and hijack sessions.

### Actors and Services

Kerberos involves three primary actors:
1. **User**: Identified by a **User Principal Name (UPN)**
2. **Service**: Identified by a **Service Principal Name (SPN)**
3. **Key Distribution Centre (KDC)**: The trusted third-party responsible for handling authentication and ticket generation
Each actor possesses a **Kerberos Key**, shared only with the KDC
	*Respectively `Kc`, `Ks` and `Kkdc` (for the KDC itself)*

Kerberos relies on three core services provided by the KDC:
1. **Authentication Service (AS)**, which verifies the user's identity (username + password).
		It issues:
        - A **Ticket-Granting Ticket (TGT)**: Encrypted using `Kkdc`, it allows the user to request service tickets afterwards
        - A **Logon Session Key**: A temporary key shared with the user for secure communication with the Ticket-Granting Service (TGS)

2. **Ticket-Granting Service (TGS)**, which issues:
        - A **Service Ticket**: Grants the user access to a specific service
        - A **Service Session Key**: A temporary key shared between the user and the service for secure communication.

3. **Client/Server (CS) Service**: The service verifies the user's authorisation using the **Service Ticket** presented by the client.

### Protocol

1. **Identification:** The user provides his identity (username+password) to the AS
2. **Authentication:** Kerberos AS looks up the database and ensures one of the user corresponds to the provided username+password. If the identity is valid, it forwards a **TGT** (encrypted with `Kkdc`) along with a **Logon Session Key** (encrypted with `Kc`).
> Now, the user is authenticated (identity was verified), but still can't access any service (as this relies on **authorisation**).

4. **Request Service Access:** The user sends the **TGT** he just obtained to the **TGS**. The TGS decrypts the TGT using `Kkdc` to validate the user’s identity.
5. **TGS Delivery:** If TGT is valid, a **Service Ticket** (encrypted with the desired service's key `Ks`) is issued, along with a **Service Session Key** which is shared between the user and the service.

6. **Service Access:** The user sends the **Service Ticket** to the target service, and the latter will decrypt it with `Ks` to check user's authorisations. The user and the service now communicate securely using the **Service Session Key**.


****
## Federated Identity Management

Kerberos is a template for Federated Identity Management.
	*Identity management scheme is shared across multiple enterprises, networks, applications...*

**Identity management** is the approach of providing users an enterprise-wide access to resources by **defining their identity** (human, device, process).
	*The core concept of this is the Single SIgn-On (SSO): User authenticates once, but can access all resources without having to authenticate on each service individually.*
