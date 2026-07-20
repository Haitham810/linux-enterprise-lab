# Distopia Network — Linux Enterprise Server Infrastructure

A fully functional enterprise-grade server environment built on Rocky Linux (server) and Ubuntu (client), configured from scratch as part of a Systems & Network Administration project.

## Overview

The Distopia Network simulates a real-world organizational infrastructure, providing core network services across a static LAN (192.168.200.0/24). All services are integrated, hardened with firewalls, and where applicable, secured with SSL/TLS encryption.

## Services Configured

| Service | Technology | Protocol/Port |
|---|---|---|
| DNS (authoritative) | BIND9 | UDP/TCP 53 |
| Mail Transfer Agent | Postfix | SMTP — TCP 25 |
| Mail Delivery Agent | Dovecot | IMAP/POP3 + SSL |
| Web Server | Apache HTTP Server | HTTP 80 / HTTPS 443 |
| Database | MariaDB | TCP 3306 (remote access) |

## Chapter Breakdown

### Chapter 1 — DNS Server (BIND)

- Installed BIND9 (bind + bind-utils) on Rocky Linux
- Configured /etc/named.conf to listen on static IP 192.168.200.4 and accept queries from the local subnet
- Created forward lookup zone (distopia.org) mapping hostnames to IPs via A Records
- Created reverse lookup zone (200.168.192.in-addr.arpa) mapping IPs back to FQDNs via PTR Records — required for SMTP sender verification
- Verified with dig from Ubuntu client (both forward and reverse resolution confirmed)

### Chapter 2 — Secure Email Server

- Installed Postfix (MTA) and Dovecot (MDA)
- Configured main.cf: set myhostname, mydomain, mynetworks, and home_mailbox (Maildir format for Dovecot compatibility)
- Generated a self-signed RSA 2048-bit certificate using OpenSSL; placed in /etc/pki/tls/
- Configured Postfix to enforce TLS on outgoing transport (smtpd_tls_cert_file)
- Configured Dovecot (10-ssl.conf) to require SSL for all client connections; disabled plaintext authentication
- Verified end-to-end by sending email between two users via Thunderbird on the Ubuntu client

### Chapter 3 — Apache Web Server

- Installed Apache and configured firewall rules for HTTP traffic
- Set up website directory structure and configured a virtual host for distopia.org
- Verified HTTP access from Ubuntu client

### Chapter 4 — MariaDB Database

- Installed MariaDB server on Rocky Linux; configured firewall for port 3306
- Created databases, users, and access permissions
- Installed MariaDB client on Ubuntu; verified remote login and table operations from the client node

## Troubleshooting Log

Real issues encountered and resolved during the project:

| Issue | Root Cause | Resolution |
|---|---|---|
| DHCP client retaining old IP | Cached lease not released | Forced lease release and renewal; cleared DHCP cache |
| FTP directory listing failed (passive mode) | Passive port range not open in firewall | Opened passive port range in firewalld |
| vsftpd failed to start | SSL certificate path mismatch in config | Corrected rsa_cert_file path in vsftpd.conf |
| Apache SSL configtest failed | Misconfigured certificate directive | Fixed SSLCertificateFile path in Apache vhost config |
| HTTPS not reachable from client | Firewall not reloaded after rule change | Ran firewall-cmd --reload to apply rules |
| Kea DHCPv4 server not running | Service not enabled on boot | Enabled and started kea-dhcp4 service |

## Environment

- Server OS: Rocky Linux (RHEL-based)
- Client OS: Ubuntu
- Network: VMware isolated NAT — 192.168.200.0/24
- Server IP: 192.168.200.4
- Client IP: 192.168.200.80
- Domain: distopia.org

## Skills Demonstrated

Linux Administration, DNS / BIND, Postfix, Dovecot, SSL/TLS, OpenSSL, Apache, MariaDB, firewalld, Network Services Troubleshooting, Rocky Linux, Ubuntu

## Author

Haitham — BSc Information Systems Security, Asia Pacific University (APU)
