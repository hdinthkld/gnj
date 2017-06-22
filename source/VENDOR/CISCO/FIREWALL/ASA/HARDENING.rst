.. _cisco_asa_hardening:

#################################
Cisco ASA Hardening Best Practice
#################################

Overview
#########

Hardening a Cisco ASA firewall falls the following key practices:

* Secure Operations
* Management Plane
* Securing Config
* Logging and Monitoring
* Through Traffic


Secure Operations
#################

* Monitor for Cisco software vulnerabilities and advisories
* Leverage AAA
* Centralise Log Collection and Monitoring
* Use Secure Protocols where possible (SSH rather than telnet for example)
* Gain Traffic Visibility with Netflow


Management Plane
################

The management plane is used in order to access, configure and manage the
device. It is used by a number of protocols (such as SNMP, SSH, FTP, Netflow,
Syslog, RADIUS, TACACS+, etc).

* Password Management
* Enable HTTPS access (up to 5 sessions)
* Enable SSH (default 1024-bit modulus)
* Configue Timeout for login sessions
* Configure encrypted passords
* Use AAA (TACACS+ or RADIUS)
* ASA Image signing (9.3 and above)
* Configure clock timezone and NTP
* Remove DHCP service is not needed
* Control Plane Access List

Securing Config
==================

* Image Verification (as of 9.1.2 and 8.4.4)
* Encrypt passwords in config
* Disable Password Recovery

Logging and Monitoring
######################

* Configure SNMP with strong (non-default) community strings
* Enable Readd Access
* Enable SNMP Traps
* Configure Syslog
* Configure Console Logging Severity
* Configure Logging Timestamps
* Configue Netflow (ASA supports NSEL or NetFlow V9)

Through Traffic
###############

* TCP Sequencce Number Randomization
* TTL Decrement
* Fragment Chain Fragmentation Checks
* Configure Protocol Inspection
* Configure Unicast Reverse Path Forwarding
* Threat Detection
* BotNet Filter
