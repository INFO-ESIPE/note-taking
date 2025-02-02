[[Information Security]]
***
**Table of Contents**
```table-of-contents
```

***
## Anonymity

### What for?

Anonymity ensures that users can interact with digital systems without revealing their identity. As it aims to protect values like privacy and freedom of speech, It encompasses:
1. **Non-identifiability**: Alice’s real identity is indistinguishable from others within a group, even when attributes are stored in a database
2. **Unreachability**: Ensures that, while an observer knows the identity, they cannot pinpoint the physical or logical location of the subject.
3. **Untrackability**: Prevents correlating multiple interactions or tracking patterns:
	- Within services: Registration or fingerprinting
	- Across services: Techniques like tracking cookies aggregate data to form profiles


> [!info]- Mind the difference between Anonymity and Pseudonymity
>**Anonymity**: Complete non-identifiability or untraceability.
>**Pseudonymity**: Use of an alias (nickname), often protected by law, for partial identity concealment.


### For who?

Even though the lectures says that it is mostly for whistleblowers, corporations, law enforcement, governments and criminals, it is good for any individual to remain as anonymous as possible in most situations.


### Data Anonymisation and K-Anonymity

Developed by Sweeney and Samarati, K-Anonymity protects individuals in datasets. A dataset satisfies this property if every record is indistinguishable from at least `k-1` others, ensuring individuals cannot be uniquely identified.


***
## Anonymisation Networks

Two ways to gain "anonymity" on our communication:
1. **Proxies**: All our communication are passed to an intermediate (the proxy) first, allowing them to appear as coming from the proxy itself (and not the end user hiding behind it),
![[ESIPE/Semestre 5 (Pavie)/Information Security/16 - Anonymity/proxy.png|550]]
> [!caution] Downside: Single Point of failure, the proxy knows both who we are and who we communicate with

2. **Mix Network (Mixes)**: Random permutation and decryption of messages.
![[mix-network.png|550]]
> [!caution] Downside: Each layer requires asymmetric encryption (costly and slow)


***
## The Onion Router (TOR)
*Useful - How TOR Works: https://youtu.be/QRYzre4bf7I?t=69

In essence, TOR leverages the strength of both mixes and proxies:
1. Proxies: Communication passes through **a series of Onion Routers** (intermediaries). The final router delivers the message to its recipient.
2. Mixes: **Symmetric encryption is established with each Onion Router** along the communication path, creating a secure route.

![[onionkeys.png|700]]


Each Onion Router is associated with a dedicated **symmetric key**, introducing the concept of **layered encryption**:
- Each message is encrypted in layers, with one layer dedicated to each Onion Router.
- As the message traverses the path, each router decrypts its respective layer and forwards the message to the next.
> ![[onion.png|500]]
 
The design ensures that no single router knows both:
- The source (the identity of the original sender)
- The destination (the identity of the final recipient)

> [!example]
> In our example above, the first router knows only the sender and the next router it must forward the communication to.
> The middle routers only knows intermediates, it doesn't disclose any information about us.
> The last router (exit node) knows the recipient but has no knowledge of the sender or previous routers.
>
> Furthermore, keep in mind that those intermediates can not necessarily deduce if the machine they interact with is another onion router or a real client/server.

Keep in mind that the communication must still use TLS, as the last node can see our message. TOR is here to anonymise our exchange, not the frame itself. 


### Attacks Against TOR
*We would need a dedicated networking class in order to explain & understand this correctly... Let's talk about them in a superficial manner though*

Here are the most common type of attacks:
1. **Traffic correlation**: If the same organisation (e.g., the NSA) possess both our entry and exit nodes, it can know who we are and who we communicate to.
2. **Malicious nodes**: As we explained above, exit nodes can read unencrypted communication if HTTPS or other encryption layers are not used.
3. **End-to-End Timing Attacks**: Timing analysis can correlate the sender's outgoing traffic with incoming traffic to the recipient.
