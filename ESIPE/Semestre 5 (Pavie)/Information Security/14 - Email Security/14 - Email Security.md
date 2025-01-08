[[Information Security]]
****
**Table of Contents**
```table-of-contents
```

****
## Architecture

In short, an email follows this workflow:
- **Author sends email:** MUA → MSA → MTA → (Multiple MTAs) → MDA → MS.
- **Recipient retrieves email:** MS → MUA.

![[mail_architecture.png]]

Now, let's take a look at the actors displayed above.
- **Message User Agent (MUA)**: Software used by the author or recipient **to compose, send and read emails**.
	*Desktop clients (Thunderbird, Outlook...) or web (gmail, protonmail...). MUAs typically rely on SMTP (for sending) and IMAP/POP3 (for retrieving emails)

- **Mail Submission Agent (MSA)**: First server in the path of the outgoing email, responsible for accepting messages from the user's MUA. 
  Usually listen on port 587 (submission) or 465 (secure submission), rather than the default SMTP port 25 (Port 25 is typically reserved for MTA-to-MTA communication).
  Verifies the sender's credentials, ensures the email structure is valid, and forwards the mail to the MTA.

- **Message Transfer Agent (MTA)**: Backbone of **email routing**. 
  Transports emails across the internet between domains, so it gets closer to the recipient. It provides error notifications in case of an undeliverable email.
  The logic is: An MTA looks up the recipient's domain via the **DNS MX records** to determine which MTA to forward the email to.

- **Mail Delivery Agent (MDA)**: Accepts the incoming messages from the final MTA and stores them in the MS for the recipient to access.
  MDAs also handle local email delivery, such as sorting emails into specific folders or applying server-side rules (e.g., filters for spam).

- **Message Store (MS)**: Database-like system for storing emails (long-term storage).
	*We'll see that the MS' behaviour changes a lot depending on if we use IMAP or POP*


For all of this to work, two more actors are involved.
- First of all, the **Administrative Management Domain (ADMD)** which represents an email provider (e.g., ISPs, enterprise mail relays). 
  They manage MTAs, allowing them to operates based on their own policies for email handling and trust decisions.

- Finally, the **Domain Name System (DNS)** which—in short—maps IP addresses to domain names.
  *IP-domain mapping is the main feature (A/AAAA records), but DNS servers also contains MX (Mail eXchange) records which maps a domain name to a mail server*

> [!info]- What an MX record looks like
> ```bash
> Domain			TTL   Class    Type  Priority      Host
> example.com.	1936	IN		MX		10         onemail.example.com.
> example.com.	1936	IN		MX		10         twomail.example.com.
> ```


***
## Protocols

**SMTP (Simple Mail Transfer Protocol)** is used to **move the mail from sender to recipient**, hopping from an MTA to another.
By using MX records provided by DNS servers, an SMTP client can contact the SMTP server to forward the email to.
The client sends an **enveloppe**, which contains a body (the mail itself) along with the sender+recipient(s) emails.
	*ESMTP (Extended SMTP) is an upgraded version of SMTP, which provides additional features (Cryptography, Host authentication)*

Two protocols can be used by the recipient to **retrieve the mail** from its MS:
1. **POP (Post Office Protocol)** is more suited for people accessing their mails from a single device, as the POP server typically deletes the mail once the recipient downloads it.
   To be precise, POP3 is the currently used version as it provides authentication.

2. **IMAP (Internet Message Access Protocol)** is a modern and more permissive alternative to POP3.
   Instead of downloading mails, the client directly connects to the server and manipulates mails from here. IMAP **maintains the email state on the server**, such as whether a message is read, flagged, or deleted. This synchronisation ensures that actions performed on one device reflect on all others.


|                          | **POP3**                                                       | **IMAP**                                                |
| ------------------------ | -------------------------------------------------------------- | ------------------------------------------------------- |
| **Storage**              | Emails are downloaded and (typically) deleted from the server. | Emails remain on the server.                            |
| **Device Accessibility** | Limited to one device (unless configured to leave copies).     | Accessible from multiple devices.                       |
| **Synchronization**      | No synchronization between devices.                            | Full synchronization across devices.                    |
| **Server Space**         | Saves server space as emails are downloaded.                   | Requires more server space.                             |
| **Internet Dependency**  | Emails are accessible offline after download.                  | Requires an active internet connection.                 |
| **Protocol Ports**       | Port 110 (unencrypted) / 995 (SSL/TLS).                        | Port 143 (unencrypted) / 993 (SSL/TLS).                 |
| **Use Case**             | Best for single-device usage with limited server storage.      | Best for multi-device usage and cloud-based management. |


***
## Formats

### IMF

The format an email used to follow was defined through **Internet Message Format**.
The message is composed of a header (key-value pairs separated by a colon `:`), and a body:
```bash
Date: October 8, 2009 2:15:49 PM EDT
From: “William Stallings” <ws@shore.net>
Subject: The Syntax in RFC 5322
To: Smith@Other-host.com
Cc: Jones@Yet-Another-Host.com

Hello. This section begins the actual
message body, which is delimited from the
message heading by a blank line.
```
> [!info]
> Like for HTTP, the header and the body are separated by a blank line


### MIME

A modern extension of IMF is the **MIME specification**, which addresses several limitations of its counterpart over SMTP:
- Cannot attach binary files (non-ASCII text overall) to the mail
- Limited to ASCII encoding (a lot of characters can't be encoded, like "é")
- Strict size restriction that rejects email if not respected
- ...

MIME fixes all of those issues. Furthermore, it ensure the message's correct structure, allow for multipart messages (plaintext, HTML and attachments within a single email).

Here is an example:
```bash
MIME-Version: 1.0
Content-Type: multipart/mixed; boundary="boundary-string"

--boundary-string
Content-Type: text/plain; charset=UTF-8

This is the plain text part of the email.

--boundary-string
Content-Type: text/html; charset=UTF-8

<html>
  <body>
    <p>This is the HTML part of the email.</p>
  </body>
</html>

--boundary-string
Content-Type: application/pdf
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="file.pdf"

JVBERi0xLjQKJ...(base64 encoded file content)...

--boundary-string--
```
> [!info]
> The `boundary-string` separates each part of the email.


### S/MIME

S/MIME (Secure/Multipurpose Internet Mail Extensions) builds on MIME to provide security features for email communication:
- **Authentication**: Ensures the sender's identity through digital signatures.
- **Integrity**: Guarantees the message has not been altered in transit.
- **Confidentiality**: Encrypts the message body so only the intended recipient can read it.
- **Non-Repudiation**: Ensures the sender cannot deny sending the email, thanks to digital signatures.

It works like this:
1. **Digital Signatures**: The sender signs the email with their private key. The recipient verifies the signature using the sender's public key.
    *Provides authentication and integrity*

2. **Encryption**: The sender encrypts the email using the recipient’s public key. Only the recipient, with their private key, can decrypt and read the email.
    *Ensures confidentiality*

S/MIME has a strong integration with major MUAs like Outlook, Thunderbird, or Apple Mail.


***
## Threats

Like most other fields, emails are no exception to those common threats:
- **Authenticity:** Unauthorised access to email systems.
- **Integrity:** Unauthorised modification of email content.
- **Confidentiality:** Disclosure of sensitive information.
- **Availability:** Preventing users from sending or receiving emails.


***
### Mitigation
*Note: We will focus on the network side. Mechanisms against malware, advanced phishing... won't be discussed here.*

### Authentication and Domain Protection

First comes the **Sender Policy Framework (SPF)**: an email validation system that detects and prevents unauthorised sources from sending emails on behalf of a domain.
It works by allowing domain owners to specify which mail servers are permitted to send emails using DNS TXT records.
> [!info]- SPF Record example
> It looks like this:
> ```bash
example.com. IN TXT "v=spf1 ip4:192.0.2.0/24 include:_spf.google.com ~all"
>```
> - `v=spf1`: SPF version.
>- `ip4:192.0.2.0/24`: Authorizes IP range.
>- `include:_spf.google.com`: Authorizes Google servers.
>- `~all`: Indicates "soft fail" for unauthorized servers.


Second procedure is the **DomainKeys Identified Mail (DKIM)**, allowing an MTA to add a digital signature to the header of outgoing emails, ensuring their authenticity and integrity.
The signature is verified using the public key published in the sender’s DNS records.
> [!info]- DKIM Record example
> A DKIM record looks like this:
> ```bash
> default._domainkey.example.com IN TXT "v=DKIM1; k=rsa; p=BASE64_ENCODED_PUBLIC_KEY"
> ```


Last mechanism for the authentication part is **Domain-based Message Authentication, Reporting, and Conformance (DMARC)**, which allows domain owners to specify how email receivers should handle unauthenticated messages.
> [!info]- DMARC Record example
> DMARC is built on top of DKIM and SPF to protect domains from unauthorised use.
> ```bash
> _dmarc.example.com IN TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@example.com"
> ```
> - `p=quarantine`: Instructs receivers to quarantine unauthenticated emails.
> - `rua`: Specifies where reports should be sent.


### DNS Security Extensions (DNSSEC)

Crucial component in securing email systems as it **ensures the authenticity and integrity of DNS data**. 
	*It prevents attackers from tampering with DNS responses, which could otherwise be used to redirect emails to malicious servers.*

DNSSEC ensures that records like SPF, DKIM, and DMARC are retrieved securely by verifying their cryptographic signatures.
It operates by signing DNS zones with private keys and enabling recipients to verify these signatures using public keys stored in the DNS.


### S/MIME Integration

As previously discussed, S/MIME is directly involved in providing end-to-end encryption and digital signatures for email communication.
- Belongs to **message-level security** mechanisms.
- Complements domain-level protections like SPF, DKIM, and DMARC by securing the email content itself, ensuring **confidentiality**, **integrity**, and **authentication**.


### STARTTLS for Secure Transport

STARTTLS is an email security protocol **that upgrades an unencrypted connection to an encrypted one using TLS (Transport Layer Security**).
- Emails are encrypted during transmission between MTAs or between an MUA and an MTA.
- STARTTLS is often used in conjunction with SMTP, IMAP, or POP3, providing encryption for these protocols.

*Purpose is to prevent man-in-the-middle attacks, and ensure privacy during transport of the mail from MTA to MTA.*


### Putting It All Together

- **SPF, DKIM, and DMARC** secure email domains from spoofing and unauthorised use.
- **DNSSEC** ensures the authenticity and integrity of DNS records critical for email security.
- **S/MIME** secures the email's message content, providing confidentiality and authentication at the user level.
- **STARTTLS** encrypts email transmission, safeguarding data between servers.

Combining these mechanisms provides a robust multi-layered defense against email threats.

> [!info]- To go further on "Trustworthy email"
> NIST has published a document providing guidelines to enhance trust in emails, which can be found [here](https://csrc.nist.gov/pubs/sp/800/177/r1/final)
