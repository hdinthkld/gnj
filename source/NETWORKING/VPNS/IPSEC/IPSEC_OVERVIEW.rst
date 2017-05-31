##########
IPSEC VPNs
##########

.. toctree::
  :titlesonly:

  IKE Version 2 <IKEV2>
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

  * Phase 2

    * Quick Mode

  * IPSEC Packet Processing

    * SPD
    * SADB


Key management is the process of generating, distributing and storing of
encryption keys. In IPSec the Internet Key Exchange (IKE) protocol is is the
default method for secure key negotiation.

In order to exchange keys over an insecure transport medium, IKE uses the
Diffie-Hellman Key management protocol.

Diffie-Hellman Key Exchange
---------------------------

The DH Key Exchange depends on the discrete logarithm problem where it assumes
that it is computationally infeasable to calculate a shared secret key given
two public values.

It works by calculating two values a generator (g) and a parameter (p). The two
communicating parties (Alice and Bob) calculate a random private value (a and b
respectively).  A public value is then derived using the previos 'p' and 'g'
values

Alice will calculate her public value as :math:`X = g^a \, mod \, p`

Bob will calculate his public value as :math:`Y = g^b  \, mod \, p`

These public values are then exchanged between Alice and Bob.

Alice then calculates :math:`k_{ab} = (g^b)^a \, mod  \, p`

Bob calculates :math:`k_{ba} = (g^a)^b \, mod \, p`

Because :math:`k_{ab} = k_{ba} = k` they now know the shared secret key
:math:`k`.

Security Association
--------------------

A security association (or SA for short) is a basic building block of IPSec.
The SA is an entry in th SA Database (SAdB) that contains information about the
security that has been negotiated between two parties for IKE or IPSec.

There are two types of SA, both of which are established between peers using
the IKE protocol:

* IKE or ISAKMP SA
* IPSec SA

The IKE SA is used for traffic that controls the VPN tunnel such as when
negotiating algorithms to use to encrypt IKE traffic and to authenticate peers.
There is only one IKE SA between peers and commonly has less traffic along with
a longer lifetime that IPSec SAs.

IPSec SAs are used for negotiating the encryption algorithms to apply for IP
traffic (the actual user data) between peers.  Each IPSec SA is unidirectional
so there will always be at least two IPSec SAs (one in each direction).  It is
also possible to have multiple pairs of SAs as each defines a unique set of IP
hosts or IP data traffic (for example one pair per source/destination network
specified).

The establishment and maintenance of both types of SAs is the major function of
the IKE protocol.  The definition of how this key exchange and negotiaion is
done is divided into IKE (RFC 2409), ISAKMP (RFC 2408), OAKLEY (RFC 2412) as
well as the ISAKMP Domain of Intepretation (RFC 2407).

Although ISAKMP and IKE are used interchangably they are different.  ISAKMP
provides the means to authenticate a peer and to exchange information for key
exchange.  IKE defines how the key exchange is done.

IKE operates in two phases in order to establish the SAs for IKE and also IPSec:

* Phase 1 provides mutual authentication of the VPN peers and establishment of
  the session key. This creates the ISAKMP SA or IKE SA using DH excahnge,
  cookies and an ID exchange.  Once established all IKE communication between
  the initiator and responder is protected with both encryption and integrity
  checks. This then provides a secure channel so that Phase 2 negotiations can
  be performed safely.

* Phase 2 provides the negotiation and establishment of IPSec SAs using either
  the ESP or AH protocols to protect IP data traffic.

IKE Phase 1 Operation
---------------------

Phase 1 can be performed in one of two modes, Main or Aggressive.  Either way
the result is the establishment of an ISAKMP/IKE SA.

The IKE SA has a number of mandatory parameters:

* Encryption Algorithm (e.g. 3DES)
* Hash/Integrity Algorithm (e.g. SHA1)
* Authentication Method (e.g. Pre-shared keys)
* Diffie-Hellman Group (e.g. Group 2)

Other parameters are optional, such as SA Lifetime and most devices will have a
default value for this if not defined (in terms of seconds or kilobytes)

Comparision of Main and Aggressive Mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. rubric:: Main Mode

Main mode communicates 6 messages between peers starting with the Initiator.

It offers identity protection and flexibility in terms of the parameters and
configurations that can be negotiated.

The exchange occurs as follows:

#. initiator sends initiator cookie and SA payload which contains the Phase 1
   parameters.
#. The responder replies with the selected parameters for each of the proposals
   along with the SA header response and the ISAKMP Header with a responder
   cookie
#. The 3rd and 4th messages occur when the keying materials are being exchanged.
   and a number of different keys are derived used for the late exchanges.
#. The 5th and 6th messages are encrypted using the keys created in the previous
   steps and authenticated using the hashes derived.

The proposals must be selected or denied in their entirety, the responder is
not allowed to pick and choose elements from different properties.

In addition in Main Mode the ID Payload is encrypted so the responder cannot
determine who it is talking to therefore when authenticating using a pre-shared
key, the identity can only be determined based on the source IP address.

.. rubric:: Aggressive Mode

Aggressive Mode completes it's exchange using only 3 messages which speeds up
the IKE transaction processing.  This speed however comes at a cost of some
security.

The 3 messages are exchanged as follows:
#. Initiator sends the ISAKMP header, security association, DH public value,
   nonce and identification ID.
#. The responder then replies with the choosen transform set for each of the
   proposals and the DH half key.  This message is authenticated but not
   encrypted.
#. The initiator then replied to the responder with the message authenticated
   so that the responder can make sure the hashes matches the one calculated.

In the past one of the primary uses for Aggressive mode was for remote access
IKE clients as the responder would have no prior knowledge of the source IP
address of the initiator and pre-shared keys were used for authentication.

Because identities are passed in the clear and DH parameters cannot be
negotiated, Agressive Mode is deemed less secure. As a hash of shared secret is
passed in clear text, if an eavesdropper was able to intercept this an offline
brute force attack could be used to obtain the clear-text secret.

In modern networks where sites are using dynamic IP addresses it is recommended
to use digitial certificates for authentication and not pre-shared keys. This
allows Main mode to be used with the identity being established from the
details in the certificate.

Authentication Methods
^^^^^^^^^^^^^^^^^^^^^^

One of the parameters exchanged as part of IKE Phase 1 is the authentication
method.  The most common methods used are pre-shared key and digital signatures.

.. rubric:: Pre-Shared Key Authentication

In this method both peers must know a shared secret in advance which is
communicated via an out-of-bands medium (such as courier or telephone call
between the security engineers). Due to the simplicity by which the keys can
be exchanged it is very widely used.

They keys used by the Diffie-Hellman exchange are derived from this pre-shared
key.

As mentioned above, when pre-shared keys are used and the VPN peers do not
have fixed IP addresses (such as in a remote access scenerio), Aggresive Mode
is the only choice for this peers.

.. rubric:: Digitial Signature Authentication

Peers are able to be authenticated with public key signaturues (either DSA or
RSA).  The Public Keys are normally obtained as Certificates and IKI allows
for the exchange of them between intitiator and responder.

The IKE Phase 1 process will exchange the public key (certificate) during the
final 5th and 6th messages of a Main Mode exchange. The signers public key
is used to decrypt and verify the messages being sent.

Various details in the certificate can be used to identify the peer so it can
be used in situations involving dynamic IP addresses.  However due to the
complexity of needing to setup a Public Key Infrastructure (PKI) this is none
used as oftern as it should be.

For more information on setting up a PKI and Certificate Authority
Infrastructure, refer to the
:doc:`PKI </SECURITY/CRYPTOGRAPHY/PKI/PKI>` section of this
journal

IKE Phase 2 Operation
---------------------

Once the IKE Phase 1 SAs have been establihed, Phase 2 can begin.  Phase 2 only
provides a single method of operation, known as *Quick Mode*. Once completed
the two peers are ready to pass traffic using either ESP or AH.

As mentioned previous, a minimum of two IPSec SAs are required for two-way
communication between IPSec peers.

Quick mode uses 3 message exchanges. All of these exchanges are protected by IKE
and  therefore encrypted and authenticated using the same keys derived in Phase
1.

The Quick Mode Process is as follows:

#. The initiator will send the ISAKMP header and IPSec SA payload (including
   proposals and transforms) that will be used for bulk data. A new nonce
   is also exchanged between the peers. Because multiple quick mode
   exchanges may be ongoing, a Message ID is included to distinquish each
   transaction.
#. The responder replied with the choosen proposal along with the ISAKMP header,
   nonce and hash
#. In the 3rd message the initiator authenticates with a further hash value.
   This is then validated to avoid the possibility of earlier packets being
   replayed.


Perfect Forward Secrecy
^^^^^^^^^^^^^^^^^^^^^^^

By default the IPSec Keys are derived from the same initial key which could lead
to an attacker with knowledge of the initial key being able to calculate all the
current and future keys until IKE renegotiates.  When Perfect Forward
Secrecy (PFS) is enabled, new DH public values will be exchanged and the
resulting shared secret will be used to generate new key material.

IPSec Packet Processing
-----------------------

The exact way in which IPSec packets are processed by a host or router is often
down to a vendor chose to implement it so the above process should be seen
as a top-level overview of the communication not exactly how vendors have
designed their various solutions.

RFC 2401 does describe a general approach that vendors should adopt to support
interoperability that achieve the functional goals of IPSec.

This model describes two databases that IPSec implementions with maintain:

* Security Policy Database (SPD)
* Security Association Database (SADB)

.. rubric:: Security Policy Database

The SPD defines various selectors to identify packets that require IPSec
Services:

* Destination IP address
* Source IP Address
* Name
* Data sensitivity level
* Transport layer protocols
* Source and Destination Ports

One or more of these selectors will define the IP traffic encompassing the
policy entry. Each entry will indicate whether traffic matching should be
bypassed, discarded or subject to IPSec processing. If to be processed
by IPSec the entries will include an SA (or SA bundle).

.. rubric:: Security Association Database

Each entry in the SADB defines the parameters associated with one SA.  Each
an IPSec SA is created, the SADB is updated with all the parameters associated
with it.

The SA entry for an inbound packet is obtained by indexing the SADB
with the destination IP in the outer IP header, SPI and IPSec security protocol
(either ESP or AH).

The SA entry for an outbound packet is obtained by a pointer to the SADB in
the SPD.

The SADB contains the following parameters:

* Sequence Number
* Sequence Number Overflow
* Anti-replay window
* SA lifetime
* Modes
* AH authentication algorithm
* ESP authentcation algorithm
* ESP encryption algorithm
* Path MTU

IPSec Enhancements
==================

* IKE Keepalives / DPD
* Idle Timeout
* Reverse Route Injection
* NAT-T
* Split Tunneling


IPSec and Fragmentation
=======================
