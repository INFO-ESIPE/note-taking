[[Information Security]]
13/11/2024
****
**Note:** It might be asked to describe the [[10 - Key Management and Distribution#Symmetric Key Distribution Using Asymmetric Encryption|second protocol]] for the exam :)
****
**Table of Contents**
```table-of-contents
```

****
## Current concern

So far we have our ciphers, but we need a reliable way to spread the keys (dispose) and to ensure identity (of both the manager of the exchange system and the key holders).
	*Those ciphers are only reliable as long as the keys are owned by the correct people. If a secret key is leaked, everything that was (or will be) done with it cannot be trusted anymore*

Major issue is that most of the **storage and exchanges are carried by proxies or middle-men** (or at least they are included in the chain at one point), which is a problem for confidentiality and integrity.

Core challenges:
- **Key Distribution**: Ensuring that cryptographic keys are shared securely between parties.
- **Key Storage**: Keeping keys safe from unauthorised access.
- **Key Lifespan**: Regularly updating keys to prevent long-term exposure to attacks.


****
## Symmetric Key Distribution Using Symmetric Encryption

- Only the two parties must see and possess the key. 
- They must be able to exchange the keys often (as we want to prevent cryptanalysts to collect too much data about our keys from available ciphertexts).
	*This is especially the case when you encrypt a huge amount of similar data (e.g., HTML documents as they share a similar structure, money transactions as they share the same structure besides the account holders and the amount of money ...)*

This last requirement actually reveals how important the **key distribution technique** is (delivering the key to the two parties only, which guarantees integrity and confidentiality)

### Procedures

Available solutions:
1. A generates a key and **physically deliver** it to B
	*Moves from entity to entity instead of going through proxies, which is good. However, this is not very convenient and quick, especially to establish an end-to-end encryption over the network. If A wants to talk to a lot of people, he will have to exchange thousands of keys (one per remote machine), which seems infeasible physically. The **distribution method is bad for large-scale exchanges***

2. A **trusted third party generates the key** and **physically deliver** it to both A and B
	*We must be sure that this proxy is trustworthy, which is an issue. Also, like the previous solution, physical distribution is bad when it involves a lot of people*

3. If A and B have previously and recently used a key, one party can **transmit the new key** to the other, **encrypted using the old key**
	*If an adversary or a proxy gained access to the key, this method becomes unsafe as all the subsequently-generated keys are visible in cleartext*

 4. If A and B each has an encrypted connection to a **third party C**, C can **deliver a key on the encrypted links to A and B**
	*Again, C must be a trustworthy proxy, but this is the most widely-used method as it offers a good **symmetric distribution system**.*

> [!fail]- Is it reliable? Not really...
> As we can see, a lot of assumptions are made (the proxy is trustworthy, the last key was never leaked ...). This is why distributing symmetric keys via symmetric keys is relatively inconvenient, as it requires a master key for every pair of people we want to help in the exchange of key. 
>
> Furthermore, a share authority solution is also dangerous, as it will end up storing all the master keys (the safe becomes pretty much like a treasure cave that every cryptanalyst or adversary will attempt to break into).*
>
> **We need to find another solution, this is not reliable enough**


****
## Symmetric Key Distribution Using Asymmetric Encryption

As we have seen [[05 - Asymmetric Ciphers|here]] , public-key encryption is way slower, which is why reserve it for secret keys encryption before distribution (and we avoid encrypting encrypting large documents etc as it's sooo slow damn...)
	*We encrypt a symmetric key via the private key of our asymmetric cipher*


### Simple Secret Key Distribution

A first naive solution would be the **Simple Secret Key Distribution**:
1. A generates a public/private key pair {`PUa`, `PRa`}
2. A transmits a message to B consisting of `PUa` and an identifier of A (`IDA`).
3. B generates a secret key (`Ks`) and transmits it to A, which is encrypted with `PUa`.
4. A can recover the secret key via it's private key, which means only A and B can know the secret key.
5. A purges `PUa` and `PRa`, and B purges `PUa`
	*We don't want to leave traces of the exchange as it gives clues (or even the entire answer) if an adversary can put his hands on it.*

> [!fail]- Is it safe? No...
> This works but introduces some issues, including identity assumptions.
>	*A must trust B, and B must trust the identifier A forwards him)*


### Mutual Authentication

Instead, we tend to use the following **mutual authentication** procedure (which is more complex but more reliable):
1. **Initiate connection:** A uses B’s public key (`PUb`) to encrypt and send a message containing an identifier of A(`IDA`) and a nonce (`N1`), which is used to identify this transaction uniquely.

2. B decrypt the message with it's private key `PRb` 

3. **A confirms identity of B:** B responds with a message encrypted with `PUa` and containing:
	- A’s nonce (`N1`). **Because only B could have decrypted message (1), the presence of `N1` in message (2) assures A that B is really B, and not an impersonator**...
	- Another nonce generated by B (`N2`).

4. **B confirms identity of A:** A returns `N2`, encrypted using `PUb`. 
	*Like the step above that proved B was B, A is proving his identity with the same technique, since only him could retrieve `N2`*

5. **A sends the secret key:** A forwards a secret key `Ks` in a nested encrypted message:
	- The inner message (`E(PRa, Ks)`) is encrypted with `PRa` to prove that A sent it.
	- The outer message (`E(PUb, ...)`) is encrypted with B’s public key to ensure that only B can read it.

6. **B retrieves the secret key:** B decrypts the outer layer using it's private key (`PRb`), then decrypts the inner layer using A’s public key (`PUa`) to retrieve `Ks`.

> [!caution]- Is it reliable? Somehow, but not ideal...
> Both parties now share the secret key (`Ks`) for further communication, and each party is assured they are communicating with the owner of the other's keypair due to the use of **nonces** and **public/private key pairs**.
>
> While this protocol authenticates the **keypair owners**, it does **not** guarantee the identity of the entities (A and B). For instance:
> 	*The only statement we can do is: Owner of keypair A can trust and communicate with owner of keypair B.*
>
> The strongest statement we can make is:  **The owner of keypair A trusts they are communicating with the owner of keypair B.**
>
> This solution is way better than relying on symmetric encryption like in the first chapter as it introduces **mutual authentication**, ensures **confidentiality** and **integrity**.
> However, it requires a trusted way to **publish and verify public keys**, which leads to the need for a **Public Key Infrastructure (PKI)**.


****
## Public key distribution

There are four ways of distributing public keys:
- [[10 - Key Management and Distribution#Public Announcement|Public Announcement]]
- [[10 - Key Management and Distribution#Public Directory|Public Directory]]
- [[10 - Key Management and Distribution#Public-key Authority|Public-key Authority]]
- [[10 - Key Management and Distribution#Public-key Certificates|Public-key Certificates]] (most widely used)


### Public Announcement

Each entity can broadcast his public key to the community at large (forums, mailing lists...)

> [!fail]- Is it safe? No... 
> Convenient, but weak: Susceptible to forgery; anyone can pretend to be you.


### Public Directory

Every owner of a keypair places his public key in a public directory that is trusted (managed by a directory authority that verified the sender's identity beforehand).
	*An authentication system (sign-in) works as well*

This looks like a simple array (key-value pairs):

| Owner   | Public key    |
| ------- | ------------- |
| goobert | 198ff81ab9... |
| forax   | 99edf9233...  |
| ...     | ...           |

> [!fail]- Is it safe? No...
> This method is more secure than public announcement, but still introduces a risk if an adversary succeed in gaining access to the directory and change the keys.
> 	This method does guarantee confidentiality, but not integrity as there is still a scenario where it's possible for someone to replace the keys and perform a MITM attack...*
> 	
> A second issue is that most of those directories are owned by governments (or large companies that have to comply with the government's orders and country laws). If the government asks the company to change the keys (for espionage or manipulation/psyop purposes), they can do it. Integrity issue once again...
> 
> Last issue we might encounter is that it can quickly become a bottleneck, as everyone will turn to the directory each time they have to acquire a public key (heavy traffic).
>	*This one is only performance-related, but it's still important to have a blazing fast system instead of a slow peace of crap*


****
14/11/2024

### Public-key Authority

This system dynamically distributes public keys upon request.
The authority encrypts the response with a dedicated **master key** (one for each request).

It works like this:
1. **Requesting the Public Key:** User A sends a request to the Public-Key Authority, asking for User B's public key.

2. **Verification by the Authority:** The Public-Key Authority checks its secure directory to find User B's public key, it then encrypts it along with additional details (like timestamps) **using its own private key**. 
	*This process is known as **signing**.*

3. **Secure Response to User A:** The authority sends the signed message back to User A. User A uses **the authority’s known public key** to **decrypt** the response and verify its authenticity.
	*If the message was indeed signed by the Public-Key Authority, it means that the public key of User B is legitimate.*

As mentioned, **it is mandatory for the requester to possess the authority's public key** to decrypt user B's public key. 

> [!question]- But how do we get all the public keys? 
> Trusted authorities' public keys are **pre-installed in most operating systems and web browsers**. Trusted **Certificate Authorities (CAs)** like "Let's Encrypt", "DigiCert", and others have their public keys embedded in a **"root certificate store,"** which is maintained and updated by the OS or browser developers.

> [!success] Is it reliable? Yes

### Public-key Certificates

A certificate is just a file provided to us by a complex structure called the **Public Key Infrastructure (PKI)**, more details [[10 - Key Management and Distribution#Public-key Infrastructure|here]].
	It is like a digital ID card issued by a CA, linking a public key to the identity of its owner (a person, an organisation, or a website).

It works like this:
1. **Certificate Issuance:** The owner (e.g., a website) generates a public/private key pair and submits the public key along with their identity information (e.g., domain name, organisation details) to a **Certificate Authority**.

2. **Validation:** The CA verifies the owner's identity through various checks (like domain ownership verification).

 3. **Signing the Certificate:** Once verified, the CA creates a **certificate** that includes:
    - The owner's public key
    - Information about the owner (like their name or website)
    - A **digital signature** from the CA, created using its own private key
    - Expiry date (validity period of the certificate)
	    *This one is used for two reasons: When certificates are renewed, they use the most up-to-date technologies (fix potential security issues from previous certificate), and also its for profit yeah*
    **This signature proves that the CA has verified the information.**
    
4. **Using the Certificate**:
    - When you visit a website (e.g., [https://www.bnpparibas.it/it/](https://www.bnpparibas.it/it/)), it sends you its public-key certificate.
    - Your browser checks the certificate’s digital signature using the CA’s public key (which is pre-installed in the browser).
    - If everything matches, you know the public key belongs to the real website, so you can safely send encrypted information.

> [!success]- Is it reliable? Yes
> Well, this is safe unless the Certificate Authority gets his private key leaked (or uses it to issue fraudulent certificates), which happened in the past:
> https://blog.mozilla.org/security/2011/03/25/comodo-certificate-issue-follow-up/


****
## X.509 Certificates

X.509 Certificates are the most widely-used schemes of public-key certificates.
	*Used in most network security applications (TLS, VPN, S/MIME)*


****
## Public-key Infrastructure

This is the system that manages the creation, distribution, storage, and verification of digital certificates and public keys. It is designed to ensure secure communication over networks by providing a framework for establishing **trust** between users, devices, and systems.
	*It encapsulates a lot of what we have described above*

**Components:**
- **Certificate Authority (CA)**

- **Registration Authority (RA)**, which acts as a mediator between users and the CA.
    It handles tasks like verifying user identities and approving certificate requests, but it does not issue certificates.

- **End Entity**, the end user who holds the certificate (e.g., a website owner) 

- **Certificate Revocation List (CRL)** and **Online Certificate Status Protocol (OCSP)**. If a certificate is compromised or no longer valid, it is added to the CRL or checked via OCSP to inform users that it should no longer be trusted.
