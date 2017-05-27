##########
IPSEC VPNs
##########

.. toctree::
  :titlesonly:

  VPN_FUNDAMENTALS
  VPN_ENHANCEMENTS
  VPN_GRE_IPSEC
  VPN_IPSEC_NAT
  VPN_IPSEC_XAUTH
  VPN_ARCHITECTURES

Introduction
============

It's been mentioned already in the overview of VPN technologies that IPSec is
not a single protocol instead a suite of protocols working together to provide
the necessary security services.

RFC 2401 defines the fundamental components of IPSec to be:

* Security Protocols
* Key Management
* Algorithms

Due to the way in which IPSec works it's not easy to understand one of these
components without understanding the offers.

Encryption is a fundamental part of various IPSec functions and the relevant
terminalogy will be mentioend as necesary.  For a more detailed understanding,
consult the chapters in this journal on
:doc:`Cryptography </SECURITY/CRYPTOGRAPHY/CRYPTOGRAPHY_OVERVIEW>`

These notes were written up as part of my studies for the CCNP SIMOS exam in
May 2017 as served as part of my revision. They are not intended to be complete
discussion of the entire IPSec Protocol stack.

.. _ipsec-security-protocols:

Security Protocols
==================

The purpose of IPSec is to provide protection for the data that is being
transmitted over the network.  Depending on the nature of the traffic a number
of services can be offered as covered previously:

* Access Control
* Confidentiality
* Data Integrity
* Authentication
* Protection against replay

IPsec provides two protocols that offer different levels of protection to the
data, these protocols are Encapsulating Security Payload (ESP) and
Authentication header (AH).

Each of these protocols again can be used along with different encapsulation
methods depending on the type of network they are being used on. The methods
of encapsulation are called Transport and Tunnel mode and will be covered first.

Transport vs Tunnel Mode
------------------------

Transport Mode
^^^^^^^^^^^^^^

Transport mode works by inserting the ESP or AH header between the original IP
header and the upper layer protocol (e.g. UDP or TCP). The IP header remains the
same except for the IP  protocol field, the IP header checksum is recalculated.

When this mode is used any routing is based on the original destination IP
address. Because of this it is most useful when traffic between two hosts
needs to be protected but becomes more difficult when moving to a site-to-site
deployment.

When combined with GRE, transport mode still has it's uses as in this instance
the GRE endpoint serves as the host endpoint.

Another limitation of transport mode is that it cannot used along NAT
translation between two IPSec peers. In the case of AH, this is because NAT
changes the source IP address in the IP header therefore it will no longer match
the AH header causing the receiving endpoint to drop the packet.

For ESP, it is possible that NAT may work but as the higher level protocol is
encrypted a NAT device cannot read more specific information (such as port
numbers) so is more limited in the type of NAT it can do.  Depending on the NAT
device in front of the VPN device it may be limited to just one device, more
advanced devices may be able to read the ESP information (such as the SA's) and
make enough sense out of them to determine the individual connections.

It is also potentially less efficient that tunnel mode due to the encryption
engine having to displace (effectively rewriting) the IP header rather than just
encrypting the entire packet.

The size of the packet will also be increased due to the additional headers
which could lead the packet being oversized for the networks supported MTU. This
however is not as much as concern as it is for Tunnel mode.

Tunnel Mode
^^^^^^^^^^^

Tunnel mode works by encrypting the entire original IP packet into another IP
datagram and an AH or ESP header is added between the outer and inner headers.

In the case of AH, the entire new packet is authenticated (excluding certain
mutable fields, such as TTL). This means that it still will not work through a
NAT gateway.

For ESP, everything except the new IP Header is authenticated and the original
packet along with ESP trailer is encrypted. Because the new IP header
is not taken account of if the packet is modifed it can pass through NAT without
issue (assuming the NAT gateway supports IPSec Passthrough otherwise same
limitates as transport mode).

The primary benefit however is that routing decisions are now based on the outer
IP header meaning it can pass over public networks using globally unique
IP addresses, keeping the inner (private) IP addresses completely hidden until
the packet reaches the other VPN gateway.

The downside to tunnel mode however is that because it includes an additional IP
header along with ESP header, trailer this increases the packets size quite
substantially which may take it over the allowed MTU of the given network.


Encapsulated Security Payload vs Authentication Header
------------------------------------------------------

IPSec offer two means by which the original packet data can be encapsulated.

Sometimes all that is required is to ensure that the data has not been modified
other times confidentiality is essential to offer as much protection as possible
that the data has not been examined en route to it's destination.

Encapsulating Security Payload (ESP)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Encapsulating Security Payload header provides all of the necessary services
for a secure VPN as covered under :ref:`ipsec-security-protocols`.

This implementation is achieved by enclosing the encrypted version of the
orignal payload inside of an ESP header and trailer.  The ESP header is
indicated by setting the upper layer protocol in the IP header to IP Protocol
50.

One of the key fields inthe ESP header is the security parameter index (SPI),
this along with the destination address and protocol in the IP header identifies
the security association to be used to process the packet.  The SPI can be used
to looking the SA in the security association database (SADB).

A sequence number is included in the header by the header and is simply a unique
increasing number used to provide replay protection services.

It should be noted that the ESP header itself is not encrypted otherwise the
remote peer would not be able to identify which connection the packet is
associated with.  It also explains why for more advanced NAT gateway's it
may be possible for ESP traffic to pass through them without issue.

Authentication Header (AH)
^^^^^^^^^^^^^^^^^^^^^^^^^^

Unlike ESP, Authentication Header doesn't provide confidentiality and replay
protection is optional.

AH sets the upper layer protocol in the original IP header to IP Protocol 51.

Because AH creates the authentication data based on the entire packet, it's
possible some of this may change in transit resulting in the authentication
failing. To combat this the following fields are zero'd out before the
authentication digest is hashed:

* ToS
* Flags
* Fragment Offset
* TTL
* Header checksum


Key Management And Security Associations
========================================

* Diffie-Hellman Key Exchange
* Security Associations and IKE Operation

  * Phase 1

    * Main vs Aggressive Mode
    * Authentication Methods
  * Phase 2

    * Quick Mode

  * IPSEC Packet Processing

    * SPD
    * SADB

IPSec Enhancements
==================

* IKE Keepalives / DPD
* Idle Timeout
* Reverse Route Injection
* NAT-T
* Split Tunneling


IPSec and Fragmentation
=======================
