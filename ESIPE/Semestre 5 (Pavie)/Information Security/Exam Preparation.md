[[Information Security]]
***
## Personal straightforward answers
*Feel free to correct them if they lack information, or are simply wrong*

1. What is "CIA" (in the context of the class obviously) ? Describe it.
The CIA triad is a model providing three key guidelines on how to protect a system (and its resources):
- Confidentiality: Data or system is **only accessible or understandable by authorised individuals**.
	*Often relies on encryption, access control or authentication.*
- Integrity: Data or system remain **accurate and unaltered during storage or transmission**.
	*Often relies on hashes, or reliable transport mechanisms (checksums, retransmission... like TCP provides)*
- Availability: Data or system is **accessible when needed**
	*Often relies on redundancy (RAID), replicas, failover mechanisms, anti-DDoS measures*

Achieving all three principles simultaneously is ideal, it is often challenging due to the complexity of real-world systems. Different contexts may require prioritising one principle over others:

Due to the various choices of real-world architecture an organisation has to make, a system can never be considered "truly secure" (fulfil the CIA triad). For the same reason, some context will be prioritising one principle over others.
	*Medical records or classified government documents will prioritise confidentiality
	Financial transaction platforms or file exchange platforms will prioritise integrity
	High-performance computing infrastructures for live medical analysis will prioritise availability*


2. What is an asymmetric cipher? Specify an overall definition, its properties and fields of application.
Unlike symmetric ciphers, public-key cryptography relies on a **pair of mathematically-linked keys**, where a key cannot be derived from the other. In other words, **what is encrypted with key A can only be decrypted by key B, and vice versa**.
Main disadvantage of symmetric ciphers was that both ends had to share a same key that serves both purposes. Now, an user can forward only one of the two keys, the public key, while keeping the private key for himself (hence the name).

It is up to the owner to determine which purpose should serve the keys.
If interested in authentication of content (similar to a digital signature), the private key should be the one responsible for encryption.
On the other hand, if you are interested in confidentiality (message sender has the public key), the private key should be the one responsible for decryption.

Public-key cryptography comes handy in several domains, including digital signatures or password-authenticated key agreement.
Keep in mind that public-key ciphers are way less performant than their conventional counterpart.


3. Define the concepts of identity and authentication. Provide authentication factors.
Identity is the set of attributes that **uniquely identifies an entity among others within a system**.
Authentication is the process of verifying that identity. It is about **ensuring you are who you claim to be**.
Typically, a basic user authentication service will verify the identity in two steps:
1. Identification step - User gives an identifier (typically, username and password) to the security system
2. Verification step - The system try to authenticate the claim the user makes

Several authentication factors exists, in four categories:
- Something you know: Knowledge-based
	*A password, a passphrase, a PIN, a personal question...*
- Something you are: Biometric attribute
	*fingerprint, retina, face*
- Something you do: Behavioural biometric
	*handwriting, voice pattern*
- Something you possess: An electronic keycard, a cryptographic key...

   
4. What is a VPN? (We haven't seen this one much on class somehow lol)
Virtual Private Networks are services that interconnects private networks over a public network (Internet) via **tunnelling**.
These tunnels encapsulate the traffic, ensuring isolation from other traffic on the public network. This process can happen either at the network layer (L3VPN) by leveraging protocols like IPSec, or at the data link layer (L2VPN) hence providing a direct connection that appears as if devices are on the same local network.

VPNs does not inherently guarantee confidentiality or encryption (e.g., VPN BGP/MPLS). Furthermore, the potential privacy that a VPN can provide heavily relies on their exit nodes' juridiction and policies.

   
5. What is a hash function?
A hash function takes an **input** `n` **of variable size** and **outputs a fixed-size** `n'` hash value (sequence of bits called a **digest**).
We expect a hashing function to follow a certain set of properties (especially cryptographic ones):
- Deterministic: Hashing a same input always produce the same output
- Efficiency: Should compute quickly, even large outputs
- Security:
	- Preimage resistance: It should be **computationally infeasible (!!!!)** to retrieve `n` (original input) given `n'` (digest)
	- Collision resistance: Although impossible to fully satisfy, the risk for two different input to produce the same hash should be minimal.
	- Unpredictability (Avalanche effect): Any small change in the input (even a single bit) should produce a drastically different hash value. A predictable pattern makes the hash function insecure, as seen in outdated functions like MD5. 


6. What is a symmetric cipher? Specify an overall definition, fields of application.
A symmetric cipher is an encryption method where the **same key is used for both encryption and decryption**. This means the sender and the recipient must share a secret key beforehand, which must remain confidential (via a secure side-channel).
This type of cipher is way faster than its asymmetric counterpart, as the same key can both encrypt the plaintext and decrypt the ciphertext.

Symmetric ciphers are ubiquitous: secure communications (TLS, Diffie-Hellman...), file encryption to guarantee confidentiality on cold storage...


7. What is a digital signature?
Digital signatures is a major cryptographic mechanism introduced by asymmetric ciphers. The purpose is twofold:
- Allow a **signatory to claim ownership of the document**
- Recipients can **check the file integrity and authenticity**

The signatory computes the hash of the document he wishes to sign (SHA-512 is a reliable algorithm for this purpose), he then encrypts this hash with his private key. 
He now publishes both the document and its signature on the internet for other people to download, along with his public key (so people can decrypt the hash).
If someone downloads the file, he can decrypt the hash and ensure it matches the hash he computed himself. If it does, the file is legitimate and was not altered.

This methodology is extremely widespread nowadays, and remain reliable as long as the signatory's private key never get compromised/leaked.


8. Give some classic patterns for attacks against communication
Various kind of attacks exists, most of the time to compromise a point of the CIA triad. Here are some of them:
- **Denial of Service (DoS)**: Flooding a system with excessive traffic to disrupt legitimate communication and make a service unavailable. If this attack is performed in an uniform manner on several machines (usually via a botnet or Control & Command server), it is called a Distributed DoS (DDoS).
- **Replay Attacks**: An attacker captures valid data (e.g., login credentials or transaction information) and reuses it later to impersonate a user or execute unauthorised actions.
- **MITM**: An attacker intercepts communication between two parties, potentially altering or eavesdropping on data without either party knowing.
- **Spoofing**: Impersonating a trusted entity to deceive users or systems. Common types include IP spoofing, DNS spoofing, and email spoofing.
- **Eavesdropping**: Passive interception of communication to gather sensitive information.

Here are some attacks related to wireless networks (as I'm not sure what the original question asks lol):
- **Accidental Association**: Unintentional connection to neighbouring networks, exposing resources.
- **Malicious Association**: Rogue devices posing as legitimate access points to steal data.
- **Network Injection**: Exploiting vulnerabilities in routing or management messages.


9. Explain what multi-factor authentication is
MFA is a security mechanism that requires a user to provide **two or more authentication factors** to verify their identity before gaining access to a system.
Those factors should be independent and of a different factor.
	*For instance, first factor being something you know (password), second being you are (fingerprint).*

MFA becomes more and more widespread as time goes by, as it reduce risks of account compromise if the first authentication method (usually a password) gets leaked. It adds an extra layer of protection.


10. Explain what a MITM attack is
A Man-in-the-Middle attack is a form of cyberattack where an adversary **intercepts and potentially alters communication between two parties without their knowledge**.

By placing himself in between the two ends, he can:
- **Eavesdrop**: Read the exchanged data in real time, compromising confidentiality
- **Modify**: Alter the content of the communication, compromising integrity
- **Impersonate**: Relay messages while posing as one or both parties, compromising authenticity

It can happen at several layers (intercept HTTP traffic or hijack sessions, DNS spoofing, ARP poisoning...). 
Usage of encryption protocols such as TLS helps mitigating the risk by securing the communication. 