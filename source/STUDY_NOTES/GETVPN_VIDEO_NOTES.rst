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

Working Example
===============

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
