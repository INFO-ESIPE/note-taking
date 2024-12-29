[[Information Security]]
03/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Fundamentals

The **plaintext** is the entity that we want to encrypt (it can be a message, a document...)
The **ciphertext** is this message after encryption
The **cipher** is a pair of **encryption** and **decryption** transformations

**Cryptography** is the science of ciphers, while **cryptanalysis** is the science behind ciphers breaking via mathematical methods.
*Cryptography + Cryptanalysis = **Cryptology***


**Bruteforce (exhaustive key search)** is an attack where an adversary attempts to discover the plaintext by trying all possible keys, until the correct one is eventually found. It is very slow, but always works at the end of the day.

On the other hand, **Cryptanalysis** is an analytical attack solely on the ciphertext. Can be performed via several approaches:
![[analysis.png]]


Cryptographic systems can be characterised along three independent dimensions:
- **The Type of Algorithm Used for Encryption and Decryption**
    - Substitution Ciphers: Replace elements of plaintext with corresponding elements of ciphertext.
    - Transposition Ciphers: Rearrange the order of elements in plaintext to form the ciphertext.
    - Product Ciphers: Combine substitution and transposition techniques for stronger encryption.
- **The Number of Keys Used in the System**
    - Single-Key: [[02 - Cryptography Basics#Symmetric Ciphers|Symmetric Ciphers]]
    - Two-Key: [[05 - Asymmetric Ciphers|Asymmetric Ciphers]]
- **The Method of Processing the Plaintext**
    - [[03 - Block Ciphers|Block Ciphers]] 
    - [[04 - Stream Ciphers|Stream Ciphers]]


****
## Symmetric Ciphers

The **secret key** allows us to **both encrypt** our plaintext **and decrypt** the ciphertext in order to retrieve our original message.
> [!Important] 
> Keep in mind that this key acts as a parameter for an encryption/decryption function. You will always get a result output, no matter the key you provide. 
> This means that you can not be sure that what you retrieve is the real plaintext, as the result you get might be a garbage value.

The secret key must be passed between sender and receiver on a secure side channel, else all the purpose is lost:
![[sidechannel.png]]
> [!Info] Keeping the algorithms private is useless. Most of the time, encryption and decryption are performed by a "Rich Client" software. It is possible to reverse engineer this software to find the algorithm (it might take time, but it will always be found). What matter is to keep the key private (as it is easier to change if it is **compromised**).


*****
## Block & Stream Ciphers

Block ciphers splits a plaintext in **fixed-size blocks** (in general, 64 or 128 bits per block), and then encrypts each block one by one.
Each block is processed separately by the key.

Stream ciphers encrypts data in a continuous stream, **one bit at a time**. It uses a statistically random keystream— generated at the very beginning of the encryption procedure based on the provided cryptographic key—to encrypt data, accounting for the security of the whole encryption.

> [!Info] 
> Stream cipher are used for small latency, **real-time applications manipulating data-in-transit**
It consumes low processing power compared to its counterpart.
On the other hand, a block cipher is slower but reliable, offering stronger encryption for **data-at-rest**.


****
## Security of ciphers

A cipher is **Unconditionally secure** if it is impossible to retrieve the plaintext no matter how much computational resources and ciphertext samples are available to the cryptanalyst. Otherwise, the cipher is **Computationally secure**.
	*Unconditionally secure ciphers are safe against Bruteforce, but "One-time-pad" is the only example that falls in this category.*


Some ciphers uses **Substitution techniques** to obfuscate the plaintext, the most popular example being the **Caesar Cipher** (padding of 3) :
- A B C D E F G Z
- D E F G H I J C


**Transposition techniques** can also be used, where we permute the positions of symbols in between blocks. The most popular example is the **rail fence** technique.
	*One step of transposition will have the same frequency as its plaintext. More operations are required to make it reliable. This is the case for **Rotor Machines** like "Enigma".*


****
## Steganography

If cryptography is placing your valuable text into a key-protected safe, steganography is to bury it in the back of your garden.

Cryptography obfuscates a message to anyone who is not allowed to see it (does not have the key), while steganography hides the existence of this message to the eyes of someone who is not the recipient.
	*Hiding a message in metadata, in printer yellow dots, in an audio sample spectrum, hiding a message in another message through "whitespace language" ...*

