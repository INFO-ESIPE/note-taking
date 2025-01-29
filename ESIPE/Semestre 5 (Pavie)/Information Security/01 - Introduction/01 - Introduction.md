[[Information Security]]
02/10/2025
****
Safety is about protecting yourself and various **assets** from any unwanted action (not malicious, just unfortunate)
	- A slippery floor
	- Losing your wallet

Security, on the other hand, is about protecting people and assets against deliberate actions (potentially malicious from an **adversary**).
	- Computer getting stolen
	- Cryptanalyst attempting to break a cipher

A **threat** is a potential for violation of a system's security. An **attack** is a deliberate assault on a system's security. Attacks can be:
- Passive attacks
	- Eavesdropping
	- Traffic Analysis (TCPdump, Wireshark ....)
- Active attacks
	- Masquerade (IP/MAC Spoofing)
	- Denial of Service (DOS/DistributedDOS)
	- Man-in-the-Middle
	- Replay attacks (An attacker eavesdrops a secure network communication, intercept it, and then delays, resend or misredict the receiver)
	- ...

It is important to define what to protect, which assets are of value and deserve to be protected by various protocols or cryptography.
	The protection of a system—and it's data—should focus on the **CIA Triad** (**C**onfidentiality, **I**ntegrity, **A**vailability).

Since a system can never be **truly secure** (fulfils the CIA triad), it is important to monitor any activity on the system via a logwatch (or similar procedure) as it offers **non-repudiation**.
	*User can not deny his actions if they are logged by the system.

