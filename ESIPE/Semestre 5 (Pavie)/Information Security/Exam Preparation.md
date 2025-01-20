[[Information Security]]
***
## Personal straightforward answers

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



7. What is a digital signature?



8. Give some classic patterns for attacks against communication



9. Explain what multi-factor authentication is



10. Explain what a MITM attack is
