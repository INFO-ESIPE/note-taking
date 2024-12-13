[[Information Security]]
12/12/2024
****
**Note:** This class is very abstract and boring
****
**Table of Contents**
```table-of-contents
```

****
## Wireless Systems

Both wireless and mobile networks offer convenience but introduce significant security challenges due to their open and dynamic nature.

Greater security risks than a wired network:
- **Channel**: Broadcast communications are easier to eavesdrop on and disrupt than physical connections.
- **Mobility**: Portable wireless devices increase exposure to attacks (if they connect to other vulnerable networks).
- **Resource Constraints**: Devices like smartphones have limited capacity to handle threats effectively (weak antivirus and very permissive system).
- **Accessibility**: Wireless devices in remote or hostile locations are vulnerable to physical tampering.

Three actors of a wireless system can end up being attack vectors:
1. **Wireless Clients**: Devices like smartphones, laptops ...
2. **Access Points**: Entity that provide the network connectivity, often targeted by attackers.
3. **Transmission Medium**: The shared, open nature of the wireless medium makes it susceptible to attacks.


***
## Wireless Network Threats

- **Accidental Association**: Unintentional connection to neighbouring networks, exposing resources.
- **Malicious Association**: Rogue devices posing as legitimate access points to steal data.
- **Ad Hoc Networks**: Lack central control, leading to security gaps.
- **MAC Spoofing**: Attackers mimic authorised device identities.
- **Man-in-the-Middle (MITM)**: Attacker intercepts communication between devices.
- **Network Injection**: Exploiting vulnerabilities in routing or management messages.

*Some of those attacks can also affect wired networks, but they are even more efficient on wireless technology (especially MITM or any kind of eavesdropping)*


***
## Mitigate Threats

- Use encryption (WPA3) to prevent eavesdropping and tampering.
- Disable Service Set Identifier (SSID) broadcasting, reduce signal strength, use directional antennas.
	*Making the wireless access point harder to access by reducing the covered area*
- Employ **IEEE 802.1X standard** for port-based network access control to prevent unauthorised devices. This requires authentication for a device to join the network, which reduces the risk of rogue machines to become backdoors (machines we introduce on the network for the sole purpose of deploying vulnerabilities)
- Only allow specific devices to access the network based on their MAC address
	*Those can still be spoofed, its just a basic precaution*
- All the generic strategies (use antivirus, change passwords often, change default password of the router...)


****
## Mobile Security

The arrival of smartphones reduced the ability for a company to tightly control the devices on the network.
	*Usually, all computers followed an onboarding policy that was standardising protection (install company's chosen antivirus, configure LDAP access...)*

Furthermore, the ability of those devices to move everywhere (and potentially on mistrustful networks) increases the risk of importing a threat into the company's network, or leaking business information to an eavesdropper.
	*Especially if the company follows a "bring your own device" policy: employee might use his phone for random things and download malwares*


****
## Mobile Security Strategy

- The company provides dedicated smartphones, enrolled in a security pipeline that set up antivirus and security policies (enforce PIN/password protection, force frequent updates, force to change password often, prevent user from downloading unnecessary applications)
- Encrypt all communications with a VPN.
- Strengthen firewall and intrusion detection/prevention rules for mobile traffic.
- Prohibit storing sensitive data on mobile devices, or use a safe (KeePass-like)
