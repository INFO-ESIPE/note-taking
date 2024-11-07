[[Web And Multimedia Technologies]]
Cordoba
****
## Computer Network Attacks

Message authentication aims to prevent unauthorised message modifications and ensure data integrity across network communications. Major threats include:
- **Disclosure (leak)** - Release of message contents to any person or entity not possessing the appropriate cryptographic key
	*Mitigation: Cryptography*

- **Traffic Analysis** - Discovery of the pattern of traffic between parties (Frequency and duration of connections, lengths of exchanged messages ...)
	*Mitigation: Anonymity (combination of techniques)*

- **Masquerade (spoofing)** - Injection of messages into the network from a fraudulent source, Fraudulent acknowledgements of message receipt
	*Mitigation: Message Authentication*

- **Content modification (forgery)** - Changing the content of a message (insert, delete, edit or transpose data in it)
	*Mitigation: Message Authentication*

- **Sequence modification** - Modification to a sequence of message between two machines
	*Mitigation: Message Authentication*

- **Timing modification** - Delay/Replay of messages, re-use a previous session or an entire sequence of messages 
	*Mitigation: Message Authentication*

- **Source repudiation** - Denial of transmission of message by source
	*Mitigation: Digital Signatures*

- **Destination repudiation** - Denial of receipt of message by destination
	*Mitigation: Digital Signature + Well-conceived protocol*


****
## Message Authentication Functions

Authentication functions serve two main purposes:
1. **Authenticator Creation**: Generates an authenticator (e.g., hash value or MAC) for the message.
2. **Verification Protocol**: Allows the recipient to verify the message’s authenticity.

The authenticator can be of several types:
- **Hash Function**: Maps messages to fixed-length hash values as authenticators.
- **Message Encryption**: Uses the encrypted message as the authenticator.
- **Message Authentication Code (MAC)**: Combines the message and a secret key to produce a fixed-length authenticator.


Encryption alone can provide some authentication; however, MAC functions offer a robust solution as they rely on a shared secret key known only to the sender and receiver.


****
## MAC Calculation and Verification

For MAC-based authentication:
1. Sender generates a MAC value by combining the message and a shared secret key K: `MAC = C(K,M)`
2. Sender transmits the message and the MAC to the receiver.
3. Receiver recalculates the MAC from the received message and shared key, comparing it to the received MAC.

If the calculated and received MACs match:
- The message is verified as authentic and unaltered.
- The message is assured to be from the expected sender.


****
## Advantages of MAC

1. MAC functions do not require reversibility, unlike encryption algorithms, which reduces vulnerability to certain attacks.
2. Less vulnerable to being broken than simple encryption.
3. Message encryption can be applied before or after MAC calculation.
4. Separating authentication (MAC) from encryption supports flexible system designs.

Even though symmetric encryption can also serve as a basic form of authentication, MAC design is more suitable for several scenarios :
- High-demand applications (e.g., broadcast messages to multiple recipients or authenticated software execution)
- Decryption-less (MACs provide efficient and selective authentication without requiring decryption.)

