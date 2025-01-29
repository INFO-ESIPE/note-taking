[[Information Security]]
09/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Why ?

Cryptography is considered a **dual use technology**.

It is difficult nowadays to protect a communication channel physically. This is especially true for electromagnetic frequencies (WiFi, Phone hotspot ...) as it cannot be isolated (pretty much like someone screaming in a room, everyone can hear it).

This said, we can understand that accessing the physical layer (OSI Layer 1) is not an issue for the cryptanalyst, but fortunately, this layer is not so important. What matters to the cryptanalyst is to retrieve what's on the Application layer (OSI Layer 7, above the Presentation layer which is responsible of cryptography), as it is the one containing the valuable data.


****
## Block Ciphers

A **message** is simply a string using a specific—and finite—encoding. This allows the message to simply be a succession of bits, while remaining understandable to a human as it can be read like a normal text.

A block cipher splits the binary string into blocks of a given size, and then encrypt it block by block. **The ciphertext obtained is the same length as the original plaintext**.
Since cryptography is based on the ability to retrieve information, each plaintext block must produce an unique ciphertext block.

> [!question] What if the message isn't long enough to fit a block? 
> If the message is too short for the fixed size block, we pad it with 0's. If its larger, we add another block, and so on.

**Reminder :**
- 2¹⁰ = 1 Kbit
- 2²⁰ = 1 Mbit
- 2³⁰ = 1 Gbit


****
## Feistel Cipher
*Useful resource:* https://www.youtube.com/watch?v=FGhj3CGxl8I

Feistel cipher/network appears more as a framework for building an encryption algorithm. It is a structure you feed with a secret key and encryption rounds, and returns you a cipher.
Our algorithm—based on this framework—will only need to have a dedicated **Round Function**, the rest of the process is proper to the Feistel network itself.

**Behaviour** *(where `n` being the number of bits per block.):*
1. You start with a block, and you split it in two (left-hand `L` and right-hand `R` parts). 
	*Each part should have an equal size (that is why the block size is always even).*

2. We take the `R` side and give it to a "pseudo-random function" of some kind called **the round function** `F`. This function takes our secret key as unique parameter, and produces a **subkey** out of it. Once `R` is combined with our subkey, we get our return value.

3. We **XOR** the output of the step above with the `L` part of the block
	*A    B    Q
	0 & 0 = 0
	0 & 1 = 1
	1 & 0 = 1
	1 & 1 = 0*

4. We swap the halves. The original `R` (pre-processed) becomes the new `L`, and the XORed `R` becomes the new `R`

5. Repeat the operation (round). Usually, an algorithm based on Feistel is doing *16 rounds* per block.

6. When the final round is finished, we concatenate `L` and `R` to obtain our ciphertext.

> [!info]- Reversible!
> As this process is linear, it can be reversed in order to obtain our decryption algorithm. The most fascinating part of this framework is the decryption process:
> You take your ciphertext block—and the secret key, obviously—and run the exact same steps as the encryption. **The encryption and decryption processes are exactly the same**, and even if our round function is not reversible! This is because a XORed value is reversible: if you XOR it again, you retrieve the original value.

If `n` is small, the block size is practicable but vulnerable to statistical analysis (weak).

This cipher uses a logic that alternates substitution and permutation.
- **Substitution**: each plaintext element or group of elements is uniquely replaced by a corresponding ciphertext element or group of elements
- **Permutation**: a sequence of plaintext elements is replaced by a permutation of that sequence


Feistel is a relatively easy algorithm to **"analyse"**. Analysing means that it is easy to understand what the algorithm is doing (the steps it follows) in order to encrypt/decrypt a message. This is different from **"cryptanalyse"** as this focuses on breaking the algorithm's produced ciphers.


****
## Data Encryption Standard (DES)

This was the most widely used encryption scheme in commercial and financial applications until **AES** appeared in 2001.

Data is encrypted on 64-bit blocks, using a **56-bit key** (BIG MISTAKE, the key is too small).
64-bit input is transformed—through 16 rounds—into a 64-bit output.
Since the same steps—with the same key—are followed for the decryption, it is reversible.
	*DES follows the Feistel network!*

Biggest security flaw of DES is the key, of which the size is too small. It can, with a modern parallel cluster, be cracked in around an hour.


As DES itself was too weak, we decided to go with Triple DES (3DES) as no other performant algorithm was yet available as a replacement solution at the time.
	*This fixes the small key problem, but 3DES is pretty slow and overall not so performant.*


****
10/10/2024
## Multiple Encryption

Encrypt data through several **various** algorithms. This **diversity** reduces the risk of a backdoor in an encryption software to gain access to the plaintext, as we go through a succession of encryption software with various implementations of various ciphers.

As mentioned previously, Tripe DES is available as an alternative to the current AES standard, where 3DES procedures are made using between 1 and 3 keys.
	*Double DES exists but is vulnerable to "meet-in-the-middle" attacks through predictive computation.*

Two-key triple encryption works as follows :
- Encrypt plaintext with key 1
- Decrypt previous output with key 2
- Encrypt previous output with key 1

Two-key triple decryption works as follows :
- Decrypt ciphertext with key 1
- Encrypt previous output with key 2
- Decrypt previous output with key 1

In a triple-key 3DES, the last step is performed with key 3 instead of key 1.


****
## Advanced Encryption Standard (AES)

Replacement of DES since 2001. The point was to be as secure as 3DES, but quicker and overall more performant.

> [!Info]- Origin of AES 
> NIST organised a cryptography tournament where each applicant had to design the most secure, performant, quick, reliable, widely-usable symmetric algorithm possible. 
> 5 of them remained in competition:
>    - **Rijndael (won and became AES)**
>    - Serpent (most secure but a little too slow)
>    - Twofish
>    - RC6 (RSA Organisation)
>    - Mars (IBM)

Block size is of 128 bits.
The key can be 128, 192 or 256 bits long. The amount of rounds depends on the length of the key (the larger it is, the more rounds).

Unlike previously-mentioned ciphers, AES decryption and encryption process is not identical. Two different software are needed if encryption and decryption are used.


****
## Mode of Operation (Cipher Operation)
*Useful resource:* https://www.youtube.com/watch?v=Rk0NIQfEXBA

We know that block ciphers take a n-bits fixed length plaintext block—and a key—and output a n-bits fixed length ciphertext block.

If the message is larger than n bits (most of the time it is, blocks are small), we have to split the message in multiple blocks, and **re-use the key** for each block.
	*Dangerous... This makes our algorithm extremely predictive and vulnerable to cryptanalysis*
The process is straightforward and convenient so far:
1. Split plaintext into blocks
2. P1 -> E(K) -> C1
	P2 -> E(K) -> C2
	...
	Pn -> E(K) -> Cn
3. C1 + ... + Cn = Ciphertext

But we would like to make it harder for a cryptanalyst to break the cipher by looking at ciphertext samples.

There are various ways of applying the block cipher. NIST designed the **(five) modes of operation** that can be used for that purpose.
	*This can be applied on any symmetric block cipher, it does not matter*


### Electronic Codebook (ECB)
*Parallel encoding (we can encode all blocks at the same time)*

This is the operation mentioned above. We encode blocks separately with the same key, and concatenate each one to obtain the resulting ciphertext

> [!failure]- Problem: ECB Penguin
> This is not secure at all as:
> - It gives away too much information to the cryptanalyst
> - If two blocks are identical, their ciphertext version will also be identical.
> This problem is well-illustrated by the **ECB Penguin**: Pixel data (colour) of our original image is encrypted and should appear completely obfuscated to us, but this is not the case at all as we can still perceive the shape of it due to pixels sharing the same colour and resulting in the same cipher block.
> ![[ecb  penpen.png]]

### Cipher Block Chaining (CBC)
*Sequential encoding of blocks (C1, then C2, etc.)*

Base is similar as **ECB**, but the following step is added to solve the aforementioned problem:
- An **Initialisation Vector (IV)** will XOR the first plaintext block with a pseudo-random value before processing to the encryption of the block
	*This IV is not a secret, it can be revealed. The only point is to avoid the situation where two identical blocks returns the same ciphertext block.*
- We take our first ciphertext block **C1**
- We XOR it with the next plaintext block **P2**
- This XOR becomes the new **P2**
> We link the output of a block with the input of the next one, and so forth...

> [!success]- Solution: ECB Penguin
> If we now try to encode our penguin, we obtain a result that does not disclose any pattern or similarity in between plaintext blocks.
> ![[correct penpen.png|275]]

Biggest issue with this method is that it is way slower.
	*We lost our parallelism feature as each block requires the output of the previous one in order to be encoded correctly.*
Furthermore, if a problem happens on a block (network issue, an adversary modify it), it impacts the following block, which will now contain gibberish value.

### Counter Mode (CTR)
*Parallel encoding

We take a base **nonce** (an integer value that is unique to this communication).
We will have `n` nonces, one for each block. Each block nonce will be incremented by one as we go forward in the blocks.
	*In other words, the algorithm will do `nonce++` for each block*

We encrypt each nonce with our algorithm and our secret key, which will produce `n` ciphertexts-version of nonces (one per block):
	base nonce = 10; 3 blocks
	nonce + 1 -> E(k) -> CN1: 1001...
	nonce + 2 -> E(k) -> CN2: 0101...
	nonce + 3 -> E(k) -> CN3: 1101...

We then XOR each plaintext block with it's ciphertext nonce:
	P1 XOR CN1 -> C1
	P2 XOR CN2 -> C2
	P3 XOR CN3 -> C3


This is the best operation out of everything.
The most important part is to **NEVER reuse the nonce** on another block
	*that's why we are doing this trivial incrementation on each block, we ensure each block nonce is unique*

> [!info]- Optional - GCM
> There is, however, a more modern version of this operation called Galois/Count Mode (GCM)—used by AES—which relies on the same principle, but applies a Galois Message Authentication Code (GMAC) that ensures nothing has been altered (similar to a hash).


****
## Galois Counter Mode (AES GCM) - Out of class scope
Note: this part is optional, this won't be in the exam at all.

Counter Mode is good at preventing a cryptanalyst from understanding our communication, but it does not guarantee than an adversary manipulated the outputted ciphertext and turned some blocks to gibberish.
We'll need some an equivalent of a **checksum/tag** that also guarantees the **integrity of our ciphertext**.
	*This way, we know that we crafted something no adversary can read (via AES), nor manipulate (via GCM). GCM is an **authenticated encryption scheme***

We will check this tag both when we encrypt or decrypt. If we see that what we obtain is different from what the tag indicates, we know that the ciphertext has been affected and that its integrity has been violated.


We want to **authenticate everything that we don't want anyone to mess with** : 
- Every ciphertext block
- The original nonce (no need to authenticate each block's nonce, only the initial value)
- Total length of our ciphertext

Each of those data will be included in our key `H`. If something is altered, the key will be different, hence it guarantees us that we can not manipulate the ciphertext — nor the nonce — without obtaining a different key.


AES-GCM is the most widely used method of encryption for web communications (and others). The only reliable alternative available is **ChaCha20-Poly1305**.
