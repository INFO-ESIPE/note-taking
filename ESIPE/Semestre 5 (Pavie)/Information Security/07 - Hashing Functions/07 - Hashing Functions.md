[[Information Security]]
Cordoba
****
**Table of Contents**
```table-of-contents
```

****
## Principle

A Hashing function **takes a variable-size block of data as input**, and **produces a fixed-size hash value**
The main purpose of hashing is **data integrity**, as any bit change on the input will result to a complete metamorphosis of the hash value
	*This integrity control is useful for :
	 - p2p exchange (making sure we assembled the segments correctly)
	 - **Store passwords** (details later on)
	 - Antivirus (database collection)
	 - **Message Authentication** - File/message transfer over an insecure network (making sure it was not altered in transit, pretty much what Gallois/Counter mode is doing with AES). In the case of an "authentication-oriented" hash, we call it a **message digest**
		 Even if the document can be transmitted on an insecure channel, this **digest ALWAYS must be transmitted on a secure channel** (for obvious reasons)
	 - Identifying copyright-protected content
	 - Detect if system files were crypted by a ransomware*


A cryptographic hash function should fulfil a few guarantees :
- Computationally infeasible to obtain collisions
	*Two different messages shares the same hash
	This risk can be — temporarily — mitigated by increasing the hash sizes the function provides, but eventually collisions will happen*

- Retrieve the original message — or elements of it — based on its hash
	*Given hash H, we can retrieve message M that has this hash*


Here are the most common and secure cryptographic hash algorithms :

| Algorithm   | Message Size | Block Size | Word Size | Digest Size |
| ----------- | ------------ | ---------- | --------- | ----------- |
| SHA-1       | < 2⁶⁴        | 512        | 32        | 160         |
| SHA-256     | < 2⁶⁴        | 512        | 32        | 256         |
| SHA-512/256 | < 2¹²⁸       | 1024       | 64        | 256         |
*MD5 is now considered insecure. We prefer to use the Secure Hash Algorithm (SHA) family*


****
## Message Authentication Codes (MACs)

Authentication of a message can be done through **Keyed Hash functions (MACs)**
A MAC is a function that takes the message and a secret key as parameters, and return a **tag (MAC value)**
	*The two parties shares the secret key*

- Once generated, the sender append the MAC tag to the original message and forwards it
- The recipient separates the MAC tag and the message, and produces his own MAC tag — via the shared secret key and the message he receives. If the MAC tag matches, the message has not been forged
	*This security is ensured as long as the attacker remains unaware of the secret key*

**MAC algorithms are generally more efficient than an encryption algorithm**


****
## Digital Signature
Useful : https://www.youtube.com/watch?v=s22eJ1eVLTU&t=120s

As explained [[05 - Asymmetric Ciphers#Fields of application|here]], we use public key encryption this way to digitally sign a file :
- The **signer (authority)** encrypt the file's hash with the private key
- The signer forwards its public key to everyone **(verifiers)** via a **Public Key Infrastructure**
- Verifiers can then decrypt the signature and compare the real file's hash with the one in the signature. If they match, the file is trustworthy.

The **signed files** are guaranteed to be from the authority, as it is the only one that has the private key
	*Of course, if the issuer's private key gets leaked, the signatures are not safe anymore...*


One could wonder why don't we simply encrypt the entire file as the signature.
The **method mentioned above guarantees authentication**. If we want to guarantee **confidentiality**, we will encrypt the message (file) as well as the hash. However, the file can be of any length, possibly several gigabytes or only one byte, which is a problem
	*If the file is large, we will obtain a large signature which is far from being convenient (bottlenecks on the network, long time for encryption and decryption via RSA). 
	If the file is extremely short (1 byte), encryption with RSA will be extremely weak and vulnerable to brute-force (as the output is the same size as the input)*


****
## Simple Hash Functions

Where n is the size of the hash (e.g. 256 bits for SHA-256) :
1. Split the file/message into blocks of n-bit each
2. Process each block sequentially : each bit position across all blocks using XOR
3. Result for each bit in hash : XOR of corresponding bits from each block


An improvement called **Rotated Bitwise XOR** exists, as it does a **one-bit circular shift (rotation)** after each block is processed 

1. Set the beginning hash value to 0
2. For each block, rotate the current hash value by one bit (left), and then XOR the block into the hash value

![[rotate.jpg]]


Aforementioned methods does not protect from a **collision** (we obtain the same hash from several different files)
	*We call the **preimage** the body we try to get the hash of. Collision is when a hash has several preimages (it is an issue, but we cannot avoid it)*

If a cryptographic hash is **collision resistant** (computationally infeasible to find a pair of preimages that shares the same hash), it is called a **Strong hash function**
