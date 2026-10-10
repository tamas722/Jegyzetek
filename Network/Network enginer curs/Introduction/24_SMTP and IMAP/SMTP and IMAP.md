What are Email Protocols (POP3, SMTP and IMAP) and their default ports?
Summarize this article with:
Last update: Sep 03, 2025
•
3 min read
Email is an essential part of business and personal communication online. The email protocols define the mechanism of the email exchange between servers and clients. This way, they allow us to send and receive messages over the network correctly.

In this article, you will find the most common Email protocols explained – POP3, IMAP, and SMTP. Each of them has a specific function and mechanism. Read on to see which configuration will best serve your particular email needs.

What are Email Protocols?
Email protocols are sets of rules that govern how emails are sent, received, and stored. The main protocols are SMTP for sending emails, and POP3 and IMAP for receiving emails. SMTP handles email transmission between servers, while POP3 downloads emails to a device and IMAP synchronizes emails across multiple devices.

Email protocols list
The standard email protocol list includes:

SMTP
POP3
IMAP
Each of them operates differently and provides a different service for managing your email account.

What is SMTP?
SMTP stands for Simple Mail Transfer Protocol, and it is responsible for sending email messages. This protocol is used by email clients and mail servers to exchange emails between computers.

How the email protocol SMTP works?
A mail client and the SMTP server communicate with each other over a connection established through a particular email port. Both entities are using SMTP commands and replies to process your outgoing emails. Thanks to the Simple Mail Transfer Protocol, messages can be sent from the same account on different email applications.

What is POP3?
How email protocol POP3 works?
The POP3 abbreviation stands for Post Office Protocol version 3, which provides access to an inbox stored in an email server. It executes the download and deletes operations for messages. Thus, when a POP3 client connects to the mail server, it retrieves all messages from the mailbox. Then it stores them on your local computer and deletes them from the remote server.

Thanks to this protocol, you are able to access the messages locally in offline mode as well.

Modern POP3 clients allow you to keep a copy of your messages on the server if you explicitly select this option.

What is IMAP?
How the email protocol IMAP works?
The Internet Message Access Protocol (IMAP) allows you to access and manage your email messages on the email server. This protocol permits you to manipulate folders, permanently delete and efficiently search through messages. It also gives you the option to set or remove email flags, or fetch email attributes selectively. By default, all messages remain on the server until the user specifically deletes them.

IMAP supports the connection of multiple users to a single mail server.

Default email ports
Email ports are communication endpoints that define how a message should be transmitted. That includes whether a message should be encrypted and exchanged securely.

To establish a connection between your email client and your mail server, you need the latter’s IP address and a port number. These attributes are assigned by IANA (Internet Assigned Numbers Authority).

Each protocol has its own port numbers to connect through and each port supports a different type of encryption.

SMTP Ports
The available SMTP ports are four and each of them underlies a different type of encryption for email sending.

25 – This port serves to send messages in plain text, although if the mail server supports it, it can be encrypted with TLS. Therefore, many Internet service providers block it, as it represents a security risk.
Port 2525 is an alternative to the SMTP port 25 and can be encrypted over TLS.
587 – This is the port IANA registered as the secure SMTP port, and it requires an explicit TLS connection. However, if the email server does not support TLS, the message will be sent in plain text.
Port 465 works over an implicit SSL connection and if the server does not support it, the operation will be aborted.
POP3 ports
Port 110 is the default POP3 port and it is not encrypted.
The encrypted port for POP3 is 995 and works over TLS/SSL.
IMAP ports
By default IMAP works on two ports like POP3:

143 – this is the default port which does not provide any encryption.
Port 993 is the secure port for IMAP and it works over TLS/SSL encryption.
What is the difference between SMTP, POP3, and IMAP?
Incoming vs. outgoing protocols
POP3 and IMAP are handling the incoming emails and they operate in different ways to retrieve or access your email messages. Thus, they are considered mail access protocols.

On the other hand, the Simple Mail Transfer Protocol is behind the message transfer from server to server, or mail client to server. As this is the protocol handling the email sending from an email account, it is labeled as the outgoing protocol.

In short, thanks to IMAP and POP3, you are able to receive emails, and SMTP allows you to send messages.

IMAP vs. POP3
As we already mentioned, both of these protocols relate to email retrieval. All modern servers support both protocols, although they function in different manners.

While the POP3 protocol assumes that your email is being accessed only from one application, IMAP allows simultaneous access by multiple clients. This is why IMAP is more suitable for you, if you’re going to access your email from different locations or if multiple users manage your messages.

On the other hand, POP3 downloads your emails to your local computer, deleting them from the server. Thus, it reduces the space your email account uses on your web server.

To sum it up, there are 3 email protocols – SMTP, POP3, and IMAP. Each of them works on specific port numbers and operates differently. If you are having trouble connecting to an incoming or outgoing server, try using an alternative port number.
