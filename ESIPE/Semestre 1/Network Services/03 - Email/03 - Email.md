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
| VRFY        | Asks the receiver to check the mailbox validity of the recipient                                           |
| EXPN        |                                                                                                            |
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

| **Second digit** | **Meaning**                                              |
| ---------------- | -------------------------------------------------------- |
| x0z              | Syntax error in the command                              |
| x1z              | Response contains additional informations (help, status) |
| x2z              |                                                          |
| x5z              |                                                          |

### ESMTP

Extended SMTP provides additional features, usage is announced at the beginning of the exchange via EHLO.
Here are some features it adds:


| **Name**   | **Role**                                                                                    |
| ---------- | ------------------------------------------------------------------------------------------- |
| SIZE       | Distant server indicates max message size it supports.                                      |
| STARTTLS   | Distant server supports TLS                                                                 |
| 8BITMIME   | Encode message content on 8 bits ASCII                                                      |
| AUTH       | Sender must authenticate itself beforehand                                                  |
| ATRN       | Authentication of the recipient required during a mail relay                                |
| CHUNKING   | Long messages can be split into chunks                                                      |
| DSN        | "Delivery Status Notification"                                                              |
| PIPELINING | Distant server supports stream of commands (no need to answer one before accepting another) |
| SMTPUTF8   | Encode email addresses and headers on 8 bits                                                |

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
## Message format

