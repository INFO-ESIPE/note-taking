[[Network Services]]
****
**Table of Contents**
```table-of-contents
```

****
## Refresher

Basics of the infrastructure are explained [[14 - Email Security#Architecture|here]].
Basics of the protocols are explained [[14 - Email Security#Protocols|here]], but we will go more in depth for SMTP in this lecture.


***
## SMTP

Simple Mail Transfer Protocol (RFC 5321) is a **stateful** protocol for sending an email from and to a server implementing the protocol. It is encapsulated in a TCP connection.
	*From MUA to MTA, or from MTA to another MTA*

Client send text commands to the server (encoded in US-ASCII).
The server responds with a message coded on a three-digit number.
	*Everything is emitted line per line*

### Commands

SMTP commands are composed of an SMTP verb, eventually followed by arguments. Answers may also be followed by arguments.
Both commands and answers are case insensitive.

A basic exchange looks like this:
```bash
MAIL FROM: <alice@gmail.com> # Command
250 OK                       # Answer
```

Here are the most common commands:

| **Command** | **Role**                                                                                                   |
| ----------- | ---------------------------------------------------------------------------------------------------------- |
| HELO / EHLO | Identifies the transmitter (contains his IP Address or FQDN).<br>EHLO used for ESMTP                       |
| MAIL        | Contains the email address of the person emitting the email                                                |
| RCPT        | Contains the email address of the recipient                                                                |
| VRFY        | Asks the receiver to verify if a mailbox exists                                                            |
| EXPN        | Asks the receiver to expand a mailing list (e.g., resolve a group email to individual recipients).         |
| DATA        | Indicate the beginning of a message.<br>End of message is represented by a line containing a single period |
| RSET        | Abandon the current transaction                                                                            |
| QUIT        | Inform the receipt that the connection will terminate                                                      |
| HELP        | Asks for information about a command                                                                       |
| NOOP        | Asks the server to return a "250 OK"                                                                       |

> [!example]- SMTP Exchange
> ![[smtp.png|650]]

### Answer codes

Each of the three digit has a specific meaning:
- Leftmost indicates the state of the request (done, pending, impossible)
- Middle indicates the answer category
- Rightmost sharpens the second digit's meaning (less important)


| **First digit** | **Meaning**                                          |
| --------------- | ---------------------------------------------------- |
| 2yz             | Command was executed                                 |
| 3yz             | Command was partially executed, need further actions |
| 4yz             | Temporary error, invite the sender to try again      |
| 5yz             | Permanent error, invalid command                     |

| **Second digit** | **Meaning**                                                   |
| ---------------- | ------------------------------------------------------------- |
| x0z              | Syntax error in the command                                   |
| x1z              | Response contains additional informations (help, status)      |
| x2z              | Connection-related issues (e.g., network or protocol errors). |
| x5z              | Mail system-related issues (e.g., mailbox or server errors).  |

### ESMTP

Extended SMTP provides additional features, usage is announced at the beginning of the exchange via EHLO.
Here are some features it adds:

| **Name**   | **Role**                                                                                     |
| ---------- | -------------------------------------------------------------------------------------------- |
| SIZE       | Distant server indicates max message size it supports.                                       |
| STARTTLS   | Distant server supports TLS                                                                  |
| 8BITMIME   | Encode message content on 8 bits ASCII                                                       |
| AUTH       | Sender must authenticate itself beforehand, explained [[03 - Email#Authentication\|later]]   |
| ATRN       | Allows authenticated clients to request mail delivery for a specific domain.                 |
| CHUNKING   | Long messages can be split into chunks                                                       |
| DSN        | Enables delivery status notifications (e.g., success or failure reports for email delivery). |
| PIPELINING | Distant server supports stream of commands (no need to answer one before accepting another)  |
| SMTPUTF8   | Encode email addresses and headers on 8 bits                                                 |

> [!example]- ESMTP Opening
> ![[esmtp.png|650]]


***
## Message structure

An email follows this structure:
1. **Enveloppe**: At the top, contains the required information for an MTA to handle the message correctly.
	*Sender & Recipient(s) email addresses, mostly*

2. **Content**: Any information for the end user. It is transmitted after the DATA command.
	In two parts:
	- **Header**: Information on the message (From, To, Cc, Subject, Message-id). Always encoded in US-ASCII.
	  One field per line, the order doesn't matter.
	- **Body**: The body of the mail itself, either lines of US-ASCII or a MIME format

![[mail-structure.png|425]]


***
## MIME

As we have explained [[14 - Email Security#MIME|here]], MIME introduces the concept of a **multipart body**  as it allows for various type of data to be included in the mail (text, HTML, binary files...).
Each fragment of the body is separated from others by a **boundary**, taking the form of one long UTF-8 string.

Each fragment should begin with a `Content-type` field specifying its nature.

An email now looks like this:
```bash
MIME-Version: 1.0
Subject: Exemple de mail avec attachement
To: carole@esipe.mlv
From: alice <alice@esipe.mlv>
Content-Type: multipart/mixed; boundary="------------PXv4TBuYpYf3sW0wH5Qem07k"

--------------PXv4TBuYpYf3sW0wH5Qem07k # UTF-8 pure text
Content-Type: text/plain; charset=UTF-8; format=flowed
Content-Transfer-Encoding: 8bit
Helloooooooo. This email contains two attached files: 
a text file and an image file
--------------PXv4TBuYpYf3sW0wH5Qem07k # HTML format
Content-Type: text/html; charset="UTF-8"
<div dir="ltr">Hiiiiiii<br></div>
--------------PXv4TBuYpYf3sW0wH5Qem07k # Attached text file
Content-Type: text/plain; charset=UTF-8; name="myfile"
Content-Disposition: attachment; filename="myfile"
Content-Transfer-Encoding: base64
Y2VjaSBlc3QgdW4gYXR0YWNoZW1lbnQK
--------------PXv4TBuYpYf3sW0wH5Qem07k # Attached image file
Content-Type: image/png; name="img-300dpi.png"
Content-Disposition: attachment; filename="img-300dpi.png"
Content-Transfer-Encoding: base64
iVBORw0KGgoAAAANSUhEUgAACDMAAAJzCAYAAAAYpcoqAAAACXBIWXMAAC4jAAAuIwF4pT92
AAAgAElEQVR4nOzde7zVdZ0v/s/ukdYc0AYdr2kYjqgEZKIIggmKQoO7PKApekr0McfRJLNz
--------------PXv4TBuYpYf3sW0wH5Qem07k-- # Final boundary ends with two more dashes
```
> [!info] Some subtypes:
> - **multipart/mixed**: Indicate presence of attached files (of any kind)
> 	*`Content-Disposition` provides information on how to display this attached file*
> - **multipart/alternative**: Each part of the body represents the same message, but in a different format
> 	*A same message can be both represented as row text (minimalist) OR as HTML (neat display)*
> - **multipart/related**: Parts are related to each other and cannot be displayed independently


***
## Trace Information

Each SMTP server that forwards a message adds its **trace information**.
There is a dedicated `Received` field for this purpose. Each MTA that was traversed adds a new `Received` field to the mail, and puts its information in it. At the end of the day, we have a list of `Received` fields, ultimately representing all traversed MTAs.

The format is the following:
- `FROM` followed by name and IP address (in between brackets) of the host
- `ID` followed by the message's local identifier
- `FOR` followed by the recipient's email address

![[trace-info.png|600]]

> [!example]- Forward message between two SMTP Servers - Role of DNS
> ![[mail-resolve.png|650]]


***
## Security

### Authentication

Ensures the SMTP client is allowed to send a message to the server. Under the hood, it relies on the **Simple Authentication and Security Layer (SASL)** protocol, defined by RFC 4422.
	*Very useful against spam, hence why its in this chapter*

Client's command looks like this:
`AUTH <mechanism> [initial-response]`

`mechanism` represents a choice between authentication methods allowed by SASL, including:
- PLAIN: login/password encoded in base64
- CRAM-MD5: challenge/response via MD5 hashing
*Both are kind of deprecated nowadays, OAuth2 and SCRAM offers stronger security.*

> [!caution]
> Never use PLAIN without STARTTLS, for obvious reasons

`initial-response` contains base64-encoded details authenticating the client.

> [!example]- Authentication via PLAIN
> Illustrated by the following:
> ```bash
> S: 220-hermes.esipe.mlv ESMTP Postfix3.4.8
> C: EHLO zeus.esipe.mlv
> S: 250-hermes.esipe.mlv Hello zeus.esipe.mlv
> S: 250-AUTH PLAIN LOGIN
> C: AUTH PLAIN AGFsaWNlAGF1IHBheXMgZGVzIG1lcnZlaWxsZXM=
> S: 235 2.7.0 Authentication successful
> C: MAIL FROM:<alice@esipe.mlv> # Careful: address specified here might not be alice's
> S: 250 OK                                    
> C: RCPT TO:<carole@esipe.mlv>
> S: 250 OK
> C: DATA
> # Etc
> ```

> [!example]- Authentication via CRAM-MD5
> Illustrated by the following:
> ```bash
> S: 220-hermes.esipe.mlv ESMTP Postfix3.4.8
> C: EHLO zeus.esipe.mlv
> S: 250-hermes.esipe.mlv Hello zeus.esipe.mlv
> S: 250-AUTH DIGEST-MD5 CRAM-MD5
> C: AUTH CRAM-MD5
> S: 334 PDQxOTI5NDIzNDEuMTI4Mjg0NzJAc291cmNlZm91ci5hbmRyZXcuY211LmVkdT4=
> C: cmpzMyBlYzNhNTlmZWQzOTVhYmExZWM2MzY3YzRmNGI0MWFjMA==
> S: 235 2.7.0 Authentication successful
> C: MAIL FROM:<alice@esipe.mlv>
> S: 250 OK
> C: RCPT TO:<carole@esipe.mlv>
> S: 250 OK
> C: DATA
> ```

### Spam

Massive submission of advertisements or scams.
Initially, SMTP servers were considered as **open mail relay**: they forward any message they receive, and don't filter possible spam emails.

What are possible protections now?
- Required authentication via `AUTH` extension
- Control sender's IP address (check if trustworthy)
- Trace information (collect all MTAs that forwarded the message)
- Collaboration with DNS services (blacklist, greylist, whitelist, SFP, DKIM)
- ISPs filtering port 25, using the ISP's SMTP server as a relay

### Blacklisting

DNSBL (DNS Black Listing) allows a SMTP server to access a list of known spam servers' IP.
If the mail it receives comes from a malicious source, the mail is destroyed.

![[dnsbl.png|600]]
> [!caution]
> This is managed by private organisations and is not normalised by IETF...

### Greylisting

We start with the following hypothesis: If our SMTP server refuses to forward the mail (with an error 4xx):
- A legitimate server will attempt to re-send it
- A spam server will not, main reason being they emphasise on volume over quality (they will focus on sending a massive load of email, and not checking the delivery for each one).

For greylisting, our SMTP server must maintain two lists:
- Trust: IPs of legitimate servers
- Blacklist: IP of identified spam servers

> [!question]- What if the message comes from an unknown server?
> - We refuse the message with a 4xx error (inviting the sender to try again)
>     - If the server sends it again, it is added to the trust list and the email is forwarded
>     - If the server doesn't respond after a certain delay, it is added to the blacklist and the email is destroyed

![[greylisting.png|600]]

### SPF

The Sender Policy Framework (SPF) checks the email sender's domain name.
The SMTP server queries the TXT record from the domain name's DNS server, and looks if the sender's IP is in it.

![[spf.png|600]]

### DKIM

DomainKeys Identified Mail (DKIM) allows to add a digital signature in the email header.

> [!example]- DKIM Usage
> Mail received by the SMTP Server:
> ```bash
> DKIM-Signature: v=1; a=rsa-sha256; s=champs; d=esipe.mlv;
> 	c=simple/simple; q=dns/txt; i=alice@esipe.mlv;
> 	h=Received : From : To : Subject : Date : Message-ID;
> 	bh=2jUSOH9NhtVGCQWNr9BrIAPreKQjO6Sn7XIkfJVOzv8=;
> 	b=AuUoFEfDxTDkHlLXSZEpZj79LICEps6eda7W3deTVFOk4yAUoqOB
> 	4nujc7YopdG5dWLSdNg6xNAZpOPr+kHxt1IrE+NahM6L/LbvaHut
> 	KVdkLLkpVaVVQPzeRDI009SO2Il5Lu7rDNH6mZckBdrIx0orEtZV
> 	4bmp/YzhwvcubU4=;
> Received: from zeus.esipe.mlv [123.4.5.6]
> by submitserver.esipe.mlv with SUBMISSION;
> Fri, 11 Jul 2019 21:01:54 -0700 (PDT)
> From: Alice <alice@esipe.mlv>
> To: Bob <bob@iut.mlv>
> # etc.
> ```
> Here, the `b=` field contains the **digital signature** of the email. The `bh=` field contains the **hash of the email body**. Those information are here to ensure the email's integrity and authenticity.
> 
> RR Requested by the SMTP Server's DNS request:
>```bash
>champs._domainkey.esipe.mlv. IN TXT
>```
