$$$$$$$$$$$$$$$$$$$$
SSL VPN Fundamentals
$$$$$$$$$$$$$$$$$$$$

SSL VPN Introduction
=============================================

Technology Overview
---------------------------------------------------
* Developed initially by Netscape
* SSL_ v1 (not released, v2, v3
* SSL_ v3.0 served as basis of TLS_ 1.0 - IETF standard
* Cisco SSL_ VPN uses TLS_

SSL VPN Mode
------------
Clientless
##########
* No need for client installation of the PC as long as they have a supported Web Browser
* Runs through the Browser
* Gateway can proxy requests from the browser to internal resources (HTTP_, HTTPS_, FTP_)

Thin Client
###########
* Designed for those non-web based applications that have static tcp port
* Uses Thin Client for supported protocols (such as Telnet, SSH_, RDP_, VNC_)
* Uses Java applets/ActiveX Plugins so clients must have Java installed on their PC
* Popup blockers can also cause problem
* Arbitary ports can be supported through the use of smart forwarding

Thick Client
############
* Requires installation of software on PC
* Not suitable for non-managed devices
* Requires Java/ActiveX installed for installation
* Provides all functionality as if user was on the LAN (assuming permitted over the VPN)
* Policies can be managed centrally


SSL VPN Connection Process
--------------------------
# Client initiated connection to server and requests a secure connection
# Client provides a list of supported encryption/integrity algorithms (Cipher suite)
# TLS_ server replies with a cipher/hash function it also supports
# Server also sends back it's identity in the form of a digital certificate that should be provided from a trusted CA of the client.  The servers public encryption key is also sent.
# Client confirms validity of certificate and generates a session key
# Client encrypts session key with the services public key and sends it to server
# Server decrypts session key and begins encrypted session

Terms Used
----------
.. _RADIUS:
  Remote Authentication Dial-In User Service
  
.. _SSL:
  Secure Socket Layer

.. _TLS:
  Transport Layer Security
  
.. _HTTP
  HyperText Transfer Protocol

.. _HTTPS
  HyperText Transfer Protocol over SSL/TLS

.. _FTP
  File Transfer Protocol
  
.. _SSH
  Secure Shell
  
.. _RDP
  Remote Desktop Protocol
  
.. _VNC
  Virtual Network Computing

