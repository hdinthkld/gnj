##################
GetVPN Video Notes
##################

Introduction
============

* Tunnel less any-to-any connectivity
* Designed mainly for MPLS
* Supported by IOS devices only, not ASA/PIX endpoints but traffic can
  pass through them.


GetVPN Overview
===============

Terminologies
-------------

* GDOI
* KS
  * KEK
  * TEK
* Rekey Process
  * Unicast
  * Multicast
* GM
* Tunnel Header Preservation
* TBAR (Time Based Anti Replay)
* COOP Servers

Advantages
----------

* Any-to-Any Connectivity of MPLS maintained
* Tunnel Less Connectivity
* VRF support
* Operates over existing routed infrastructure

GDOI
----

* Group Domain of Interpretation
* Application layer protocol
* UDP 848
* Used for Key Management, set of crypto keys and policies shared amongst a
  set of devices

Group Member
------------

  * IOS device configured to talk to a Key Server and be part of a GetVPN group
  * Performs the actual encryption and decryption of user traffic
  * Registers with KS and downloads the policies
  * Only Phase 1 policies are configured on the GM
  * Phase 2 policies are downloaded from the GM

Key Server
----------
* Group Members establish GDOI connectivity to the Key Server when they
  start up
* Responsible for creating, maintaining and refreshing the security keys
* Changes to the policies (P1, P2, ACL) or traffic to be encrypted will be distributed
  by the Key Server to the group members
* Key Encryption Key and Traffic Encryption Key are controlled by the Key Server

Key Encryption Key
^^^^^^^^^^^^^^^^^^

* Protects GetVPN Control Plane Traffic
* Used between KS and GM when policies or TEKs need to change
* Equivalent to the Phase 1 SA

Traffic Encryption Key
^^^^^^^^^^^^^^^^^^^^^^

* Protects Enterprise traffic
* Used between group members
* TEK for the group is distributed by the KS to all group members
* KS controls when the TEK needs to change and notifies GMs when this occurs
* Equivalent to the Phase 2 SA

Rekey Process
-------------

* Process by which the KS distributes keys to all group members
* Can use Unicast or Multicast

Unicast Rekeying
^^^^^^^^^^^^^^^^

* KS delivers a rekey message to all GMs individually
* GM will acknowledge the receipt on this rekey
* KS will resend the message 3 times, GM will be removed if not response is
  received after this.

Multicast Rekeying
^^^^^^^^^^^^^^^^^^

* A single rekey message is sent out to a group of devices
* All GMs will be a member of this group and they join at time of registration
* GMs will not send any acknowledgement.

Tunnel Header Preservation
--------------------------
* No additional IP header is attached to GetVPN encrypted traffic
* Original packet is encrypted and same IP header is attached
* Traffic is routed to destination using the original IP addressing


Time Based Anti-Replay (TBAR)
-----------------------------
* Protects traffic against duplicate packets
* At time the keys and policies are refreshed a timestamp is also communicated
* GM will add this timestamp to each packet
* Receiving GM will check this timestamp against countdown timer and only
  accepts the packet if they match.
* No need to synchronise time between GMs as the timestamp is communicated by the
  KS only.

Cooperative (COOP) Servers
--------------------------

* Provides redundant Key servers to the GetVPN group
* Operate in Primmary/Secondary modes
* GMs are configured with multiple KS, if no response
* KS will consider itself to be the secondary at bootup and if no primary
  found will promote itself to Primary
* Can also configure priorities of KS's to ensure consistency
* Up-to 8 KSs can be defined as COOP

GetVPN In Practice
==================

KS Configuration
----------------

* Phase 1
* Phase 2
* Encryption Algorithm
* Hashing Algorithm
* KEK Lifetime
* TEK Lifetime

GM Configuration
----------------

* Phase 1 Configuration
* Remaining configuration is downloaded from Key Server

Registration
------------

* GM is configured with Phase 1 configuration
* GM boots up and registers with KS
* GM Receives Phase 2 policies form the KS
* Sending GM sends packet to destination, encrypted with TEK according to Phase 2
  policies
* Encrypted packet still maintains original IP header
* Receiving GM decrypts packet using same TEK and policies
* Any changes to the policy will cause an immediate rekey to the GMs (either
  unicast or multicast)

NAT Considerations
------------------

* When the KS is behind a NAT gateway, it should be configured to publish it's
  NAT'd addresses so that the GMs know where to communicate with.


GetVPN Configuration Steps
==========================

Assumptions are made that all basic IP connectvity is already in place

Basic Unicast Rekeying with PSK Authentication
==============================================

Key Server Configuration
^^^^^^^^^^^^^^^^^^^^^^^^
#. Authenticate and Enroll the KS with the CA (if using RSA signatures)

#. Define Phase 1 Policy

#. Pre-shared key (if using PSK authentication)

#. Define IPSec Transform-set

#. Define ACL that define the interesting traffic for the entire GetVPN group

   * Usually Routing protocol and management traffic does not need to be
     encrypted so these should usually be excluded
   * GDOI (UDP 848) should also be excluded from encryption

#. Define the GDOI Group

   * Identity
   * Server local
   * Rekey Type, lifetime, authentication, retransmission
   * Phase 2 SA details (Profile, ACL, replay counter, address)

6. Generate RSA Keys

Group Member Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Authenticate and Enroll the KS with the CA (if using RSA signatures)
#. Phase 1 Policy
#. Pre-shared key (if using PSK authentication)
#. Define GDOI Group

   * same ID number as on KS
   * Server Address of KS
   * Authentication method (PSK or RSA)
#. Define Crypto map
   * Set Group


#.  Bind Crypto Map to Interface

Multicast Rekeying with PSK Authentication
==========================================

All Devices (including MPLS nodes)
----------------------------------
#. Enable Multicast Routing

::

  ip multicast-routing

#. On LAN/WAN Interface, enable appropriate PIM mode

::

  interface <type/slot>
    ip pim {dense-mode | sparse-mode}

#. for PIM Sparse define the ACL and apply to the SSM range

::

  access-list standard <id-or-name>
  ! Define ACEs

  ip pim ssm range <multicast-acl>

Key Server
----------

#. Complete basic setup
#. Exclude PIM and IGMP traffic from encryption
#. Define Multicast ACL
#. Define GDOI group but set to use multicast and specify the ACL

::

   crypto gdoi group <name>
     no rekey transport unicast
     rekey address ipv4 <multicast-acl-name>

Group Member
------------

#. Complete basic setup, no additional steps needed
#. Join GM to the Multicast group

::

  ip igmp join-group <multicast-ip> source <local-ip>




COOP and GM Authorisation
==========================================

Key Server
----------
#. Complete basic configuration on both primary and secondary Key Servers

  * Set the redundancy 'priority' on the secondary to a lower value than on the
    primary
  * Set the peer and key server address on each key server to their own IP
     addresses

#. Generate RSA Keys on Primary Key Server and export
#. Import the Primary Key Servers keys into the secondary

Group Member
------------

#. Complete Basic Information
#. Configure with both Key server address

Verification
=============

Basic GDOI Configuration
::

  show crypto gdoi

* Will also show the redundancy status if configured
