# Distopia Network — DNS and Secure Email Server Lab

Full write-up of the DNS (BIND) and Secure Email (Postfix + Dovecot) labs from the Distopia Network project: an enterprise-grade server environment built on Rocky Linux (server) and Ubuntu (client) as part of a Systems & Network Administration course. See the [repository README](../README.md) for the project overview, environment summary, and troubleshooting log.

## Introduction 

Reliable, secure networks are vital in today’s digital world for organizations’ daily business operations. This project will focus on developing an enterprise grade server environment using Rocky Linux for the server environment and Ubuntu as the client nodes. The server environment will be known as the “Distopia Network” and will provide necessary network services which include Domain Name Service (DNS), Secure E-mail (SMTP / IMAP), Web hosting (HTTP / HTTPS), and relational database management (MariaDB).

OBJECTIVES  
The main objective of this project is to show how to apply SNA principles into practice by building an entire, fully functional, secure and integrated server environment. The key components that need to be delivered include:

  - **Identity Management**: Creating a custom domain (distopia.org) and setting up a master DNS server.

  - **Communication**: Deploying a mail server using Postfix and Dovecot with the communication secured through SSL / TLS encryption.

  - **Web Presence**: Providing a web presence securely from within an organization using the Apache HTTP Server.

  - **Data Integrity**: Providing a relational database system accessible from remote clients.

  - **Security Hardening**: Simulating real world security standards across all of these services using firewalls, access control, and encryption protocols.

## Chapter 1 — DNS Server (BIND)

### DNS Server Configuration (BIND)

To set up a stable Domain Name System (DNS) with BIND (Berkeley Internet Name Domain) on Rocky Linux server. This service resolves our group’s Fully Qualified Domain Names (FQDNs) like cypunkserver.disto­pia.org and cypunk­client.disto­pia.org and their corresponding IP addresses. It will be the foundation for other services like Email and Web.


### Installation of BIND

The first step involved installing the necessary BIND software packages (bind for the server and bind-utils for testing tools like dig).

![](media/image1.png)

We installed the bind and bind-utils packages via the DNF package manager. The bind package provides the named service, which listens for DNS queries on Port 53, while bind-utils provides essential diagnostic tools required for verifying name resolution.

### Main Configuration (named.conf)

We configured the primary BIND configuration file to define our network environment and trusted clients.

![](media/image2.png)

![](media/image3.png)

The /etc/named.conf file was modified to listen on the server’s static IP (192.168.200.4) rather than just the localhost. We also configured the allow-query directive to accept DNS requests from our local network range (192.168.200.0/24), ensuring our Ubuntu client can utilize the server for resolution.

Furthermore, two zone definitions were added:

1.  **Forward Zone (distopia.org):** Maps human-readable names to IP addresses.

2.  **Reverse Zone (200.168.192.in-addr.arpa):** Maps IP addresses back to names (critical for the Email Server setup later).

#### Forward Lookup Zone Configuration

This database file contains the specific "A Records" for our infrastructure.

![](media/image4.png)

We created a forward zone file located at /var/named/fwd.distopia.org.db. This file contains the Start of Authority (SOA) record, which defines global parameters for the zone, and the Name Server (NS) record. Crucially, we defined "A Records" for cypunkserver (192.168.200.4) and cypunkclient (192.168.200.80), establishing the authoritative link between our hostnames and their static IPs.

#### Reverse Lookup Zone Configuration

This file handles the IP-to-Name translation.

![](media/image5.png)

A corresponding reverse zone file was configured to handle Pointer (PTR) records. This configuration allows the system to query an IP address (e.g., 192.168.200.4) and return the hostname (cypunkserver.distopia.org). This is a mandatory requirement for many secure network services, particularly SMTP (Email), to verify sender identity.

**Firewall & Permissions**

To make the service accessible and secure, we adjusted file permissions and firewall rules.

### Service Verification (Server-Side)

Before testing the client, we confirmed the service was active and syntax-free.

![](media/image6.png)

The named-checkconf utility was used to validate the syntax of our configuration files. Upon verification, the named service was started and enabled to launch on boot. The system status confirms the service is active and binding correctly to the configured network interfaces.

### Client-Side Verification (Ubuntu)

![](media/image7.png)

![](media/image8.png)

Verification was conducted from the Ubuntu Client (cypunkclient). Using the dig utility, we successfully performed a forward lookup, which resolved cypunkserver.distopia.org to 192.168.200.4. We also performed a reverse lookup on the server's IP, which correctly returned the FQDN. This confirms that the DNS server is fully operational and correctly integrating with the client node in the network infrastructure.

## Chapter 2 — Secure Email Server (Postfix + Dovecot)

Email servers are a critical component of modern communication systems, enabling the exchange of messages over the internet. At their core, email servers function as digital post offices, handling the sending, receiving, and storage of emails for users. The two main types of email servers are the Mail Transfer Agent (MTA), which manages the sending and routing of emails, and the Mail Delivery Agent (MDA), which is responsible for local delivery to users' mailboxes.

In this project, we will focus on setting up a secure email server using Postfix as our MTA and Dovecot as our MDA. Postfix is renowned for its speed, reliability, and robust security features, making it an ideal choice for handling outgoing emails. On the other hand, Dovecot provides an efficient and secure way for users to access and retrieve their emails using protocols like IMAP and POP3.

With the increasing importance of secure communication, our project will also involve configuring SSL/TLS to safeguard email transmissions against unauthorized access, ensuring that sensitive information remains private. This comprehensive approach will not only facilitate efficient email management but also enhance the overall security of email communications.

**Installation of Email Services**

### Installing Postfix and Dovecot

![](media/image9.png)

![](media/image10.png)

To implement the email infrastructure, we installed **Postfix** (SMTP server) and **Dovecot** (IMAP/POP3 server) using the DNF package manager. Postfix was selected for its performance and security as a Mail Transfer Agent (MTA), while Dovecot serves as the Mail Delivery Agent (MDA) to allow clients to retrieve messages.

#### Postfix Main Configuration

The following edits were made to the /etc/postfix/main.cf file using the anno editor

![](media/image11.png)

![](media/image12.png)

![](media/image13.png)

![](media/image14.png)

![](media/image15.png)

We configured the /etc/postfix/main.cf file to define the server identity.

  - **myhostname/mydomain:** Set to distopia.org to ensure emails are sent from the correct domain.

  - **mynetworks:** Configured to 192.168.200.0/24 to allow our local Ubuntu client to send mail without authentication errors (Open Relay for local subnet only).

  - **home\_mailbox:** Set to Maildir/ format. This creates a directory in each user's home folder to store emails, which is required for Dovecot compatibility.

#### Firewall & Service Start

We need to open the mail ports (SMTP=25) and start the service.

![](media/image16.png)

We set up the system firewall to allow traffic on TCP Port 25, which is used for SMTP. The Postfix service is configured to start automatically on boot, and it has been checked to ensure it is actively ready to handle outgoing email requests.

![](media/image17.png)

We used the command ss -tulpn | grep 25 to verify whether the SMTP service is running and listening for incoming connections on port 25. By using the ss command, we can gather detailed information about the sockets in use, and the pipe to grep allows us to focus specifically on the relevant port. This is essential for ensuring that our email service is operational and capable of sending emails.

#### Dovecot (The Mailbox)


**Step 1: Editing the Main Protocol Config**

![](media/image18.png)

![](media/image19.png)

We disable the plaintext\_auth to enhance the security of our email communication. This forces the use of SSL for email authentication, it helps to protect sensitive information from being transmitted in plain text over the internet. SSL improves our email system’s security thereby maintaining user trust and achieving the Secure Email certification marks

![](media/image20.png)

We set up Dovecot to use the Maildir format (located in \~/Maildir) to match our Postfix setup. This way, Dovecot can access the emails stored in the user's home directory. We also activated the IMAP and POP3 protocols in the dovecot.conf file and adjusted the authentication settings in 10-auth.conf to allow plain and login methods. To enhance security, we disabled unencrypted plaintext authentication, ensuring that all communications are encrypted.

#### Configure SSL/TLS (The Security Layer)

We need to create a digital ID (Certificate) and a secret Key. We will place them in the standard Linux security folders so Postfix and Dovecot can find them easily.

**Generating the certificate:**

![](media/image21.png)

We need to tell Dovecot to use the new keys, therefore we edit the Dovecot SSL config.

We edit the ssl\_cert and ssl\_key to match the location of our files. We need to ensure that ssl is set to required.

![](media/image22.png)

![](media/image23.png)

Security is a critical component of the email infrastructure. We utilized OpenSSL to generate a self-signed RSA 2048-bit certificate (mail.crt) and private key (mail.key). These were placed in the standard Linux PKI directory structure (/etc/pki/tls/).

We configured Postfix (main.cf) to enforce TLS for outgoing mail transport using the smtpd\_tls\_cert\_file directives. Simultaneously, Dovecot was configured (10-ssl.conf) to require SSL for all client connections (ssl = required). This ensures that all authentication and data transfer between the client (Thunderbird) and server occurs over an encrypted channel.

### Client Configuration & Verification (with Thunderbird)

To prove that the email server actually works, we use our Ubuntu Client to send an email from Mohammed to Hiba (2 users we created to verify the functionality of the server.) using Thunderbird.

**Installation of Thunderbird;**

![](media/image24.png)

Once Thunderbird is installed, we launch it and begin setting up one of the two accounts we made previously.

For the account setup, we keep it simple.

![](media/image25.png)

It is important we configure manually to include our server and protocols

We select the hostname to match our fqdn, the username often comes with an “@domain” but we should remember the username we set, in this case we erase that and are left with the name only. Then we go ahead and click done.

![](media/image28.png)

A window pops up warning about the illegitimacy of the site, and since we’re using a self-signed certificate and not one issued by a recognized Certificate Authority, this is not a valid SSL and is expected.

![](media/image29.png)

We override Thunderbird settings by adding a security exception, and permanently storing the exception.

We repeat the security exception process for the outgoing port as well.

![](media/image31.png)

Proof that the message has been delivered successfully from Mohammed to Hiba is available in the folder.

![](media/image32.png)

