[[Information Security]]
16/10/2024
****
## Randomness

We call random any sequence of value we cannot forecast the logic of.
Pseudorandom = deterministic.
Randomness (Random bits stream) are generated via various strategies :
- Via algorithms :
	- Pseudorandom Number Generator (PRNGs)
	- Deterministic random bit generators (DRBGs)
- Based on physical sources that produces — to our knowledge — true random outputs (lava lamp, mouse movement, atom movement ...):
	- True Random Number Generator (TRNGs)
	- Non-deterministic random bit generator (NRBGs)

Various standards and guidelines are here to help building a decent algorithm, and for specific contexts.
	*The standards for cryptographic randomness are way higher than others.*
We call a sequence "random" if :
- It is **uniform** (the odds for the next bit to be "0" are the same as the odds of being "1")
	*Chi-square test*
- It is **independent** (you can not predict the next no matter how large your base sample is)
	*Not really possible to test this behaviour correctly. You can make up your own test, but even if they pass, it is never a guarantee that your NG is independent.*

*Note that those conditions are hard to achieve, and most NGs can not fully satisfy them.*


We use PRNGs for SSH/RSA/Session keys, nonces for handshakes, and — most importantly for this class — for stream encryption. 


A **seed** is — supposed to be — a true random fixed value, mostly used as a base for pseudorandom algorithms. If this value is found — when used in a PRNG — then the sequence becomes predictable and exploitable.

**TRNGs** samples a physical source of true randomness (entropy source) and converts it to a binary stream.
**PRNGs** produces a bit sequence based on a seed and a deterministic algorithm.
	*As the seed needs to be really random, it is — most of the time — generated by a TRNG beforehand*
A **Pseudorandom function (PRF)** creates a single value (fixed size), unlike a PRNG which purpose it to generate a sequence of values.
	*PRF are used for generation of nonces, keys ... as they are finite values*

It is important for a PRNG to be :
- **Forward unpredictable:** If the seed is not known, we can not guess the incoming values even though we are aware of previous values
- **Backward unpredictable:** We should not be able to guess the seed based on a sample of the sequence it generated


****
## Stream Ciphers

You feed a key to a PRNG (called here a **Key Stream Generator**), and it will generate a pseudorandom sequence of values.
You XOR this sequence of the sequence of plaintext bytes and you obtain the ciphertext byte stream.

As mentioned in the previous class, since we use XOR, the decryption process is the same as the encryption one — as long as we use the same key and components.


The most common Stream Cipher is **RC4**.
This algorithm is quick and simple, but several security flaws were found in it (making it very unsafe).
	*RC4 was used in the WEP Wi-Fi authentication protocol, which inherited from RC4's security issues*

IETF published Request For Comments (RFC) 7465 to prohibit the use of RC4 in TLS exchanges.