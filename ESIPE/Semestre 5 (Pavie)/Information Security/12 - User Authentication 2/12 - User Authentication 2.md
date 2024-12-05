[[Information Security]]
04/12/2024
****
**Table of Contents**
```table-of-contents
```

****
## Kerberos

Kerberos is an authentication protocol based on **symmetric encryption**. It allows a **user** to access resources given by **service providers** securely over a network.

It operates in **distributed client/server infrastructures**, where one or more **Kerberos servers**handle authentication procedures.

To ensure reliability, Kerberos is designed to resist the following types of attacks:
- **impersonation**: Adversary gains access to a victim's machine (or credentials) and pretends to be him
- **Spoofing**: Adversary modifies the IP/MAC address of their machine to masquerade as a legitimate one
- **Hijacking**: Adversary intercepts valid tokens or credentials during communication to replay them and hijack sessions.

### Actors and Services

Kerberos involves three primary actors:
1. **User**: Identified by a **User Principal Name (UPN)**
2. **Service**: Identified by a **Service Principal Name (SPN)**
3. **Key Distribution Centre (KDC)**: The trusted third-party responsible for handling authentication and ticket generation
Each actor possesses a **Kerberos Key**, shared only with the KDC
	*Respectively `Kc`, `Ks` and `Kkdc` (for the KDC itself)*

Kerberos relies on three core services provided by the KDC:
1. **Authentication Service (AS)**, which verifies the user's identity (username + password).
		It issues:
        - A **Ticket-Granting Ticket (TGT)**: Encrypted using `Kkdc`, it allows the user to request service tickets afterwards
        - A **Logon Session Key**: A temporary key shared with the user for secure communication with the Ticket-Granting Service (TGS)

2. **Ticket-Granting Service (TGS)**, which issues:
        - A **Service Ticket**: Grants the user access to a specific service
        - A **Service Session Key**: A temporary key shared between the user and the service for secure communication.

3. **Client/Server (CS) Service**: The service verifies the user's authorisation using the **Service Ticket** presented by the client.

### Protocol

1. **Identification:** The user provides his identity (username+password) to the AS
2. **Authentication:** Kerberos AS looks up the database and ensures one of the user corresponds to the provided username+password. If the identity is valid, it forwards a **TGT** (encrypted with `Kkdc`) along with a **Logon Session Key** (encrypted with `Kc`).
> Now, the user is authenticated (identity was verified), but still can't access any service (as this relies on **authorisation**).

4. **Request Service Access:** The user sends the **TGT** he just obtained to the **TGS**. The TGS decrypts the TGT using `Kkdc` to validate the user’s identity.
5. **TGS Delivery:** If TGT is valid, a **Service Ticket** (encrypted with the desired service's key `Ks`) is issued, along with a **Service Session Key** which is shared between the user and the service.

6. **Service Access:** The user sends the **Service Ticket** to the target service, and the latter will decrypt it with `Ks` to check user's authorisations. The user and the service now communicate securely using the **Service Session Key**.


****
## Federated Identity Management

Kerberos is a template for Federated Identity Management.
	*Identity management scheme is shared across multiple enterprises, networks, applications...*

**Identity management** is the approach of providing users an enterprise-wide access to resources by **defining their identity** (human, device, process).
	*The core concept of this is the Single SIgn-On (SSO): User authenticates once, but can access all resources without having to authenticate on each service individually.*
