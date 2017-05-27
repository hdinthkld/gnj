######################################
Fundamentals of IPSec and Cryptography
######################################

IPSec Introduction
==================

* Secure communication over insecure network
* Enables reuse of existing connections for both public (i.e. Internet) and private  netwok services.
* Used over any public network to ensure communication between two peers is kept secret.
* Commonly used over the Internet where it may be cost prohibitive or not feasible to provide private lines (e.g. leased lines) between all sites
* Can also be used in highly secure environments to protect traffic from snooping even when over a private network, such as when using a 3rd party provider :term:`MPLS`.

.. _ref-ipsec:

IPSEC
-----

Can operate in the following modes:
 * IKEv1 / IKEv2
 * Site-To-Site or Lan-To-Lan
 * Remote Access
 * DMVPN
 * GetVPN (GDOI)


IPSec VPN Features
==================

This section covers the following VPN Features
 * Confidentiality
 * Integrity
 * Authentication
 * Anti-Replay Protection

Confidentiality
---------------

  * Encrpyption Algorithms (e.g. AES, 3DES, ECC)
  * Plain text turned into cipher text
  * Only parties with the encrypted keys can see the unencrypted data

Integrity
---------

  * Hashing Algorithms (MD5, SHA1, SHA2 Family)
  * Hashing algorithm is a one way function that turns arbitary data into a fixed size value (hash product) representing the original data.
  * Used to determine if data has been modified in transit
  * Modified data (after comparing sent/received hash) should be rejected

Authentication
--------------

  * Data Origin Authentication
  * Happens before data exchange to validate each party is who they say they are
  * Using Pre-Shared Keys and/or Certificates (via PKI)
  * Parties authentication by sending a known text encrpyted (or signed a with PKI), if this fails to validate the transer should be rejected.

Anti-Replay
-----------

  * Any data that arrives late, should not be trusted
  * Can be set in terms of data tranferred (e.g. Kilobytes) or time (e.g. seconds)

IPSec Protocol Functions
========================

IPSec is a suite of different protocols, each providing a different function to enable IPSec to work.

The following protocols covered:

* IKE
* ESP
* AH

IKE
---

* Internet Key Exchange
* Management protocols that provides a framework to exchange parameters and properties between VPN peers
* Provides the "Phase 1" element of the VPN to initally setup the encryption keys used to actually transfer the real data.
* Operates in either Main Mode or Aggresive Mode (IKEv1 only)

Main Mode
^^^^^^^^^

* 6 data exchanges between peers
* Proposals are exchanged first between peers (1-2)
* Keys are exchanged next using DH Protocol (3-4)
* Final authentication occurs and Security Associations are created (5-6)

Aggressive Mode
^^^^^^^^^^^^^^^

* Only uses 3 data exchanges between peers
* First exchange includes both proposals and key
* Responder will authenticate proposal and sends own proposal
* Initiator authenticates the session
* *What are the risks of this?*

Quick Mode
^^^^^^^^^^

* Part of IPSec Phase 2
* Quick Mode is  used to recheck the attributes between peers
* Makes use of an SPI (Security Parameter Index)
* The SPI is used to identify which peer they are communicating with
* The SPI is included within each packet sent between peers

AH
---

* Authentication Header
* Provides Integrity features but no confidentiality
* Itegrity, Authentication, Anti-Replay
* Does not provide confidentiality features
* Provides the "Phase 2" element of VPN to offer guarantees that data has not een modified in transit.

ESP
---

* Encapsulated Security Payload
* ESP Provides all of the needed features of a secure VPN
* Integrity, Authentication, Anti-Reply
* Confidentiality (Encryption)
* Provides the "Phase 2" element of VPN to transfer data securely

IPSec Phases
============

Phase 1
-------

* A single IKE bi-directional tunnel is created
* Single key is used to authenticate the session
* Used with both main mode and aggresive mode
* The type of VPN determines whether to use Main or Aggresive mode

=============  ==========
VPN Type       Mode
=============  ==========
Site-To-Site   Main
Remote Access  Aggressive
DMVPN          Main
GetVPN         Main
=============  ==========

* This needs to be checked as Remote Access Can be used with both Main/Aggressive, athough Aggressive may be the default*

Phase 1.5
---------

* Optional Step to provide additional authentication step
* Known as XAUTH or "Mode Config"
* Can be used to send additional attributes to the client, such as in remote access situation.

Phase 2
-------

* Only completed once Phase 1 is complete
* Creates 2 seperate uni-directional tunnels, one in each direction
* Firewalls must allow traffic inbound and outbound between peers (most stateful firewalls, will do this automatically based on the IKE exchange)

ISAKMP
======

* ISAKMP is used for Key excahnge by IKE
* Runs by default over UDP 500

IPSec Modes
===========

Transport Mode
--------------

* Protects Layer 4 and above layer data
* Used by DMVPN
* ESP/AH Header is added in between Layer 3 and Layer 4 headers
* Additional Layer 3 Header is added containing publically routable addresses
* Real IP Address details are not protected

Tunnel Mode
-----------

* Protects Layer 3 and above layer data
* Used by Site-To-Site, Remote Access and GetVPN
* ESP/AH Header is added before original Layer 3 Header
* Additional Layer 3 Header is added containing publically routable addresses

Security Association
====================

* Group of security parameters and policies that are agreed between two IPSec Peers
* Contains following components
  * SAD
  * SPD

SAD (Security Association Database)
-----------------------------------

* Peer IP
* SPI (Security Parameter Index)
* IPsec Protocol information (e.g. ESP/AH)

SPI (Security Parameter Index)
------------------------------

* Hash of Security Policy Database (Enc, Inte, Mode, Lifetime)
* The SPI is used by the receiving device to determine which policy in which to handle the received packets


SPD (Security Policy Database)
* Contains all the Encryption, Hash, IPSec and Lifetime information

DH Group
========

* Allows two parties to share secret key over an insecure channel
* Devices add a random value to the key which is then exchanged to the other party
* The same value calculated before is then added to the received value
* Values are then sent again, if values match then both parties know they have the same key
* The random calculated values (nonce) are never sent over the link


Encryption
==========

* Mathematical algorithm
* A key applied along with the algorithm makes the contents computationally difficult to be read by someone without the key to descript it.

Symmetric Encryption
--------------------

* Secret Key Cryptography
* Single key used to encrypt and also decrypt the data
* DES (56-bit key)
* 3DES (3 x 56-bit key process)
* AES (128-bit to 256-bit)


Asymmetric Encryption
---------------------

* Public Key Cryptography
* One key (public key) is used to encrypt the data, the second (private) key is used to decrypt the data
* Only the recipient should know the private key
* Can also be used for signing data by signing the original data with the private key, the public key can be used to validate that the data came from the real sender.
* Digitical Certificate
* RSA Signature

AH
===

* Proviates Integrity, Authentication and Anti-Reply
* IP Protocol 51
* Includes external IP Header for ICV
* Doesn't include external IP headers TTL when calculating hash
* Doesn't work via NAT as when the NAT'd packet is received, the hash values will mismatch

ESP
===

* Provides all the featues of AH (I, A, AR) as well as confidentiality (encryption)
* IP Protocol 50
* Doesn't include external IP header for ICV
* Works through NAT by taking advantage of NAT-T over UDP 4500 (by default)
* NAT-T Adds a UDP (Layer 4) header to allow intermediate devices to identify each individual VPN connection because of the unique Source/Destination Ports

NAT-T
=====

* Enables IPSec VPN sessions to pass through a NAT device
* Adds a UDP header before the ESP header so that NAT can be performed

NAT-T Steps
-----------

The following3 steps are performed in order the VPN peers to determine if NAT is in place:
* Support
* Detection
* Decision

In a bit more detail:

* IKE Phase 1 two peers exchange their vender id and IOS verion to determine the NAT-T types supported
* A hash is exchanged between peers, if a match occurs assumed no NAT otherwise it is assumed NAT is in use
* In IKE Phase 2 the UDP header is inserted before the ESP header

IKE Versions
============

IKE Version 1
-------------

* Uses 6 messages (or 3 for AH)
* Uses ISAKMP
* Has NAT-T support added as an additional feature
* Fire-Forget
* No VoIP Support
* No Cryptography mechanism for proposal exchange
* Subject to DoS attack due to the fire-forget approach

IKE Version 2
-------------

* 4 - 6 messages (4 compulsory)
* NAT-T Support built in
* Check Peer existance via cookies
* VoIP Support
* Uses Suite B Cryptography


Hashing Introduction
====================

* A one way process uses to ensure integrity
* A fixed size value is calculated by the Hashing algorithm regardless of the size of the original data
* Any difference in the hash indicates that data has been modified and therefore lost integrity so should be dropped
* MD5, SHA1, SHA2
* Older algorithms such as MD5 and SHA1 are subject to hash collision issues
