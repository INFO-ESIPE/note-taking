[[Information Security]]
17/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Principle

Asymmetric cryptography (A.K.A. public-key cryptography) uses two separate keys. **One key of the pair can not be computed from the other**.

This is a big difference compared to symmetric *(conventional)* encryption. In the later, we had to trust everyone who had the key, as this key had the power to both encrypt and decrypt information. 
Asymmetric encryption allows us to only forward a **public key**, and keep the **private key** secret.

Public-key encryption is not natively more secure than symmetric encryption at all. It is not "superior" to the other in any way. 
	*Its security relies mostly on the key's length and the computation required for breaking the cipher.*


****
## Concerns at the time

The concept of public-key encryption emerges as an attempt to solve two of the biggest concerns of cryptography (which were effective under symmetric encryption):
- **How to share the key to the two parties**
	*Historically, in symmetric encryption, there was only two ways to share a key:
	1. Meet in person and give the key: big waste of time, not convenient
	2. Trust a third party that distributes keys (key distribution centre): golden mine for hackers, not secure and hardly reliable*

- **Digital signatures**
	*We need a way to have an equivalent to signatures on official documents, which could guarantee us the authenticity of a file. We could be certain that it was provided by the correct person, and that it was not altered*


****
## Public Key Cryptosystems

Works on a **keypair**:
- **Public key**: This one can—and should—be shared publicly
- **Secret Key**: This one should remain unknown to everyone else

> [!important]
> Algorithm just generates **two mathematically-linked keys**. It is up to you to chose which key does what, and which one is public or private.
> 	*If I encrypt with key A, I can only decrypt with key B
> 	If I encrypt with key B, I can only decrypt with key A

Each key can have the opposite role depending on what we want to achieve:
- **Private Key for encryption** if you are interested in **Authenticity**
	*Similar to a signature. Only the trusted authority that possesses the private key can encrypt messages.*
- **Private Key for decryption** if you are interested in **Confidentiality**
	*Everyone can encrypt messages, only the true recipient can decrypt them and obtain the plaintext*

We call the **keyring** the set of public keys you possess in order to communicate with other people via asymmetric encryption.
	*Usually, each person will forward you its public key, so you end up with several of them...*


****
## Fields of application

Public-key cryptosystems can be used for:
- Message encryption/decryption
	*We encrypt with the public key. Only the person with the private key can retrieve the plaintext.*
- Digital Signature (in theory)
	*We sign with the private key, and people decrypt with the public key.*
- Key Exchange
	*Two sides cooperate to exchange a session key, which is a secret key for
	symmetric encryption generated for use for a particular transaction (or
	session) and valid for a short period of time*


****
## Requirements

(Difficult to satisfy) Requirements for a public-key cipher:
1. It must be computationally **easy** for a party **to generate a key pair**
2. It must be computationally **easy** for a sender A, knowing the public key and the
	message to be encrypted, M, **to generate the corresponding ciphertext**
3. It must be computationally **easy** for the receiver B **to decrypt the resulting
	ciphertext using the private key** to recover the original message
4. It must be computationally **infeasible** for an adversary, knowing the public key `PUb` **to
	determine the private key** `PRb`
5. It must be computationally **infeasible** for an adversary, knowing the encrypting public key `PUb` and a ciphertext `C` **to recover the original plaintext without the private key**

### One-way function

We call a **one-way function** one that guarantee that the **calculation of the inverse is infeasible to compute**, while the normal calculation of the function is trivial.
	*Y = f(x) is easy
	X = f⁻¹(Y) is infeasible*

### Trap-door

Now, a **trap-door one-way function** is easy to calculate in one direction and infeasible to calculate in the other direction **unless certain additional information are known** 
	*like if there was a backdoor implemented in the algorithm that allowed to retrieve information if the correct hidden parameter is provided.* 

**It is invertible under certain conditions**.
	*Y = fk(X) easy, if k and X are known
	X = fk⁻¹(Y) easy, if k and Y are known
	X = fk⁻¹(Y) infeasible, if Y is known but k is not known*

Public-key cryptosystems heavily rely on the aforementioned types of functions.
	*For Diffie-Hellman exchange, each actor selects a prime number. The value obtained by multiplying both is non-reversible (we cannot retrieve the prime numbers). However, if we know one of two two prime number that was used in the multiplication, we can retrieve the other one.*


****
## Cryptanalysis

Like conventional ciphers, public-key ciphers are vulnerable to **brute-force attacks**. To make this slower, we use very large keys (around 2 Kb of size).
	*Way slower than symmetric ciphers*

Another type of attack is **trying to compute the private key based on the public key**. 
	This was never mathematically achieved so far, but it might happen one day.*

A third type of attack—which is very unique to public-key ciphers—is the **probable-message attack**. It is very similar to a rainbow table attack.
	*We have the public key. We encrypt a lot of messages with this public key to obtain the ciphertext. If the ciphertext we try to retrieve is identical, we found the plaintext without the need of the private key.*
> It is possible to mitigate this attack by **appending a random nonce to the message**, in a similar way we use a **salt** for hashing.


****
## RSA

Most popular and widely used public-key cipher.