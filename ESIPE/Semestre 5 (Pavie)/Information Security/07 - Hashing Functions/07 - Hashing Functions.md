[[Information Security]]
Cordoba
****
**Table of Contents**
```table-of-contents
```

****
## Principle

A Hashing function **takes a variable-size block of data as input**, and **produces a fixed-size hash value**.
	*Can be expressed as: `{0, 1}* → {0, 1}n`.*

A cryptographic hash function should fulfil a few guarantees :
- **Correctness**: Deterministic
	*Hashing the same input always produces the same output*
- **Efficiency**: Efficient to compute
- **Security**: 
	- One-way-ness (preimage resistance)
		*Given hash H, we can not retrieve the original message M that outputs this hash*
	- Collision-resistance (Two different inputs shouldn't result in the same hash)
		*This only works in theory. In fact, the mere fact that a hash has a specific size makes this impossible. A collision will necessarily occur if we hash `hashSize + 1` files. We only expect the algorithm to limit the scenario as much as it can...*
	- Random/unpredictability (no predictable patterns for how changing the input affects the output)
		*This behaviour is also called “random oracle” assumption*

Since changing 1 bit in the input causes the output to be completely different, a very common use case of hashing is to ensure **data integrity**.

> [!info]- Domains where integrity control is useful
> This integrity control is useful for :
> 	- p2p exchanges (making sure we assembled the segments correctly)
> 	- **Storing passwords** (details later on)
> 	- Antivirus (malicious files database collection)
> 	- **Message Authentication** - File/message transfer over an insecure network (making sure it was not altered in transit, pretty much what Gallois/Counter mode is doing with AES). In the case of an "authentication-oriented" hash, we call it a **message digest**. Even if the document can be transmitted on an insecure channel, this **digest ALWAYS must be transmitted on a secure channel** (for obvious reasons)
> 	- Identifying copyright-protected content
> 	- Quickly detect if system files were encrypted by a ransomware*

Here are the most common and secure cryptographic hash algorithms:

| Algorithm   | Message Size | Block Size | Word Size | Digest Size |
| ----------- | ------------ | ---------- | --------- | ----------- |
| SHA-1       | < 2⁶⁴        | 512        | 32        | 160         |
| SHA-256     | < 2⁶⁴        | 512        | 32        | 256         |
| SHA-512/256 | < 2¹²⁸       | 1024       | 64        | 256         |
> The (in)famous "MD5" algorithm is now considered insecure. We prefer to use the Secure Hash Algorithm (SHA) family.


****
## Message Authentication Codes (MACs)

Authentication of a message can be done through **Keyed Hash functions (MACs)**
A MAC is a function that takes the message and a secret key as parameters, and return a **tag (MAC value)**
	*As this works on symmetric ciphers, the two parties must share the secret key.*

1. Once generated, the sender append the MAC tag to the original message and forwards it
2. The recipient separates the MAC tag and the message, and produces his own MAC tag —via the shared secret key and the message he receives.
3. If the tags matches, the message has not been forged
	*This security is ensured as long as the attacker remains unaware of the secret key*

**MAC algorithms are generally more efficient than an encryption algorithm.** More details [[08 - Message Authentication Codes (MACs)|in the dedicated lesson]].


****
## Digital Signature
Useful: https://www.youtube.com/watch?v=s22eJ1eVLTU&t=120s

As explained [[05 - Asymmetric Ciphers#Fields of application|here]], we use public key encryption this way to digitally sign a file:
- The **signer (authority)** encrypt the file's hash with the private key
- The signer forwards its public key to everyone **(verifiers)** via a **Public Key Infrastructure**
- Verifiers can then decrypt the signature and compare the real file's hash with the one in the signature. If they match, the file is trustworthy.

The **signed files** are guaranteed to be from the authority, as it is the only one that has the private key.
	*Of course, if the issuer's private key gets leaked, the signatures are not safe anymore...*

> [!info]- Why don't we encrypt the entire file?
> One could wonder: Why don't we simply encrypt the entire file as the signature?
> Well, the method mentioned above guarantees **authentication**. If we want to guarantee **confidentiality**, we will have to encrypt the message (file) as well as the hash, yes. However, the file can be of any length, possibly several gigabytes or only one byte, which is a problem...
>   If the file is large, we will obtain a large signature which is far from being convenient (bottlenecks on the network, long time for encryption and decryption via RSA). 
>   If the file is extremely short (1 byte), encryption with RSA will be extremely weak and vulnerable to brute-force (as the output is the same size as the input)


****
## Simple Hash Functions

Where `n` is the size of the hash (e.g. 256 bits for SHA-256):
1. Split the file/message into blocks of `n`-bits each
2. Process each block sequentially: each bit position across all blocks using XOR
3. Result for each bit in hash: XOR of corresponding bits from each block


An improvement called **Rotated Bitwise XOR** exists, as it does a **one-bit circular shift (rotation)** after each block is processed:
1. Set the beginning hash value to 0
2. For each block, rotate the current hash value by one bit (left), and then XOR the block into the hash value

Aforementioned methods does not protect from a **collision**.
	*We call the **preimage** the body we try to get the hash of. Collision is when a hash has several preimages (we cannot avoid this issue)*
If a cryptographic hash is **collision resistant** (computationally infeasible to find a pair of preimages that shares the same hash), it is called a **Strong hash function**
