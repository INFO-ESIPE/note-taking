[[Information Security]]
17/10/2024
****

Asymmetric *(public-key)* cryptography uses two separate keys. **One key of the pair can not be computed from the other**.
This is a big difference compared to symmetric *(conventional)* encryption, as we had to trust everyone who had the key, as this key had the power to both encrypt and decrypt information. Asymmetric encryption allows us to only forward the public key (that encrypts), and keep the decryption key hidden.

Public-key encryption is not natively more secure than symmetric encryption at all. It is not "superior" to the other in any way. 
	*Its security relies mostly on the key's length and the computation required for breaking the cipher.*
It is important to note that each has different field of application.


****
## Concerns at the time

The concept of public-key encryption emerges as an attempt to solve two of the biggest concerns of cryptography — which were effective under symmetric encryption :
- **How to share the key to the two parties**
	*Historically, in symmetric encryption, there was only two ways to share a key:
	1. Meet in person and give the key - big waste of time, not convenient
	2. Trust a third party that distributes keys (key distribution centre) - golden mine for hackers, not secure and hardly reliable*

- **Digital signatures**
	*We need a way to have an equivalent to signatures on official documents, which could guarantee us the authenticity of a file, that ensure that it does provide from the correct person, and that it was not altered*

Diffie and Hellman solved those two problems in 1976.


****
## Public Key Cryptosystems

Works on a **keypair** :
- **Public key** : This one can — and should — be shared publicly
- **Secret Key** : This should absolutely remain unknown to everyone else

Each one can have the opposite role depending on what we want to achieve:
- Private Key for encryption if you are interested in **Authenticity**
	*Similar to a signature. Only the trusted authority that possesses the private key can encrypt messages.*
- Private Key for decryption if you are interested in **Confidentiality**
	*Everyone can encrypt messages, only the true recipient can decrypt them and obtain the plaintext*

Some algorithms — such as RSA — allows any of the key pair to have a role.
	*If i encrypt with key A, i decrypt with key B
	If i encrypt with key B, i decrypt with key A
		It's up to you to chose which one is the private key, and which one is the public one that you can share. What is generated is just a pair of key, none is explicitly the private or the public one*


We call the **keyring** the set of public keys you possess in order to communicate with other people via asymmetric encryption.


****
## Fields of application

Public-key cryptosystems can be used for :
- Message encryption/decryption
	*We encrypt with the public key. Only the person with the private key can retrieve the plaintext.*
- Digital Signature
	*We sign with the private key, and people decrypt with the public key.*
- Key Exchange
	*Two sides cooperate to exchange a session key, which is a secret key for
	symmetric encryption generated for use for a particular transaction (or
	session) and valid for a short period of time*


****
## Requirements

Hard to satisfy — Requirements for a public-key cipher :
1. It must be computationally easy for a party to generate a key pair
2. It must be computationally easy for a sender A, knowing the public key and the
	message to be encrypted, M, to generate the corresponding ciphertext
3. It must be computationally easy for the receiver B to decrypt the resulting
	ciphertext using the private key to recover the original message
4. It must be computationally infeasible for an adversary, knowing the public key "PUb" to
	determine the private key "PRb"
5. It must be computationally infeasible for an adversary, knowing the public key "PUb"
	and a ciphertext "C" to recover the original plaintext


We call a **one-way function** one that guarantee that the calculation of the inverse is infeasible to compute, while the normal calculation of the function is trivial.
	*Y = f(x) is easy
	X = f⁻¹(Y) is infeasible*

Now, a **trap-door one-way function** is easy to calculate in one direction and infeasible
to calculate in the other direction — exactly like a one-way function — **unless certain additional information is known** (like if there was a backdoor implemented in the algorithm that allowed to retrieve information if the correct hidden parameter is provided). 

**It is invertible under certain conditions**.
	*Y = fk(X) easy, if k and X are known
	X = fk⁻¹(Y) easy, if k and Y are known
	X = fk⁻¹(Y) infeasible, if Y is known but k is not known*

Public-key cryptosystems heavily rely on these functions.
	*For Diffie-Hellman exchange, each actor selects a prime number. The value obtained by multiplying both is non-reversible (we cannot retrieve the prime numbers). However, if we know one of two two prime number that was used in the multiplication, we can retrieve the other one.*


****
## Cryptanalysis

Like conventional ciphers, public-key ciphers are vulnerable to **brute-force attacks**. To make this slower, we use very large keys (around 2 Kb of size).
	*That's why we mostly use public-key ciphers to encrypt small-sized messages (for instance, a **session (ephemeral) key**). Those ciphers are way slower than symmetric ciphers...*


Another type of attack is **trying to compute the private key based on the public key**. However, this was never mathematically achieved so far, but it might happen one day.


A third type of attack — which is very unique to public-key ciphers — is the **probable-message attacks**.
This one is very similar to a rainbow table attack.
	*We have the public key. We encrypt a lot of messages with this public key to obtain the ciphertext. If the ciphertext we try to retrieve is identical, we found the plaintext without the need of the private key.*

It is possible to mitigate this attack by **appending a random nonce at to the message**, in a similar way to a **salt** for hashes protection.


****
## RSA

Most popular and widely used public-key cipher.