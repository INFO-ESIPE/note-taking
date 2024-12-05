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
- **Something you do** (Any dynamic biometric): Handwriting characteristics, Voice pattern... 


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
