###################
GetVPN Fundamentals
###################

.. _cisco_getvpn_overview:

Group Encrypted Transport VPN Overview
======================================

Cisco GetVPN is a means of providing scaleable secure connectivity across
private WANs (such as MPLS) whilst also maintaining the any-to-any connectivity
of those networks.

.. rubric:: Benefits

* Highly scaleable any to any mesh topology
* Maintains network intelligence of MPLS networks
* Helps ensure low latency and jitter by enabling full-time, direct
  communications between sites.
* IP Address preservation enables encrypted packets to still carry the
  original source and destination addressing.
* Two servers can be configured Cooperative Mode (COOP) in order to provide
  fault tolarance.

.. rubric:: Limitations

* Only IKEv1 is supported between COOP servers
* IKEv2 can be used between GMs but EAP is not supported for authentication
* NAT is not supported
* Port ranges cannot be used as classifiers
* End-To-end PMTU does not work in GetVPN

Support Platforms
-----------------

.. rubric:: Software Requirements

* IOS 12.4(15)T8 and 12.4(22)T2
* IOS-XE 12.2(33)XNC

.. rubric:: Key Server Platforms

* Cisco 3845
* Cisco 7200

.. rubric:: Group Members

* Cisco 881
* Cisco 1811, 1841
* Cisco 3845
* Cisco 7200
* Cisco ASR 1004



.. _cisco_getvpn_functional_components:

GetVPN Functional Components
=============================

The following components are required for GetVPN to function:

* GDOI
* Key Server (Ks)
* Group Member (GM)

Group Domain of Interpretation
------------------------------

GDOI is a protocol that is used for Group Key and SA management.  It uses
ISAKMP for authenticating the Group Members (GMs) and Key Servers (KSs).

GetVPN only supports time-based SA expiry as it does not have any information
on the amount of traffic sent between peers.

Key Servers
-----------

The Key server (KS) has the responsibility of maintaining policies for the group,
authenticating group members (GMs) and providing the session keys for
encrypted traffic.

A key server will authenticate the GMs at the time of registration.  Only
after this is successful can a GM particate in the group.

The key server is not involved in the encryption of traffic between peers. In
affect the Phase 1 IKE SA is done between the GMs and KS, whilst the actual
Phase 2 IPSec SA is done directly between participating GMs.

The key server will perioidically refresh the SAs and notify all participating
group members prior to the expiry of the old SA.  This can be done using
either unicast or (for greater scaleability) multicast.

Two types of keys are distributed by the Key Server.  The Key Encryption Key
(KEK) is used to securely rekey messages between the KS and GMs. Whilst the
Traffic Encryption Key (TEK) is used to secure the traffic between GMs.

Group Members
-------------

A Group Member (GM) registers with the key server in order to obtain the IPSec
SA necessary to participate in the group. When registering the group member
provides the group ID to the key server and then obtains the appropriate policy
and keys for that group.

GetVPN Configuration
====================

Key Server
----------

The following minimal components should be configured on the Key Server:

* IKE Policy
* RSA Key for re-keying
* IPSec Policies
* Traffic Classification ACL

The IKE policy used as the authentication method between KS and GMs.  Whilst
pre-shared keys can be used, digital certificates are preferred for scaleability
and greater security.

The RSA Key is used to secure the re-key messages.

Matching IKE Policy and authentication methods need to be configured on the KS
and GMS in order for a successful registration.

The IPSec Policies are used to secure the data traffic and are pulled from
the KSs by the GMS at time of registration.

The classification ACL is used to determine what traffic should be encrypted
and that which should be excluded. Any changes to the ACL on the key server will
result in a rekey occuring so that clients can obtain the new policy.

Group Member
------------

The group member needs to be configured with the following:

* IKE Policy
* GDOI Policy
* Interface Config

The IKE Policy should match what has been configured on the Key Server to
avoid problems with registration.

The GDOI policy identifies what group the GM wishes to join and also the KS
server to connect to.

Once the GDOI policy is configured, it should be included in a crypto map so
that it can be bound to the WAN interface.


Verification
============

Key Server Commands
-------------------

.. code-block:: none

  show crypto gdoi
  show crypto gdoi ks
  show crypto gdoi ks acl
  show crypto gdoi ks coop
  show crypto gdoi ks members
  show crypto isakmp sa

Group Member Commands
---------------------

.. code-block:: none

  show crypto gdoi
  show crypto gdoi ipsec sa
  show crypto gdoi gm
  show crypto gdoi gm acl
  show crypto isakmp sa
  show crypto ispec sa


Troubleshooting
===============

The following commands can be used to clear existing SA and/or reset any
statistics counters:

.. code-block:: none

  clear crypto gdoi [<group>]
  clear crypto sa
  clear crypto isakmp

When troubleshooting an issue the following debug commands can be run to gain
further insight into the issue:

.. code-block:: none

  debug crypto gdoi
  debug crypto gdoi gm
  debug crypto gdoi ks
  debug crypto isakmp
  debug crypto ipsec

A Number of syslog entries will also be recorded starting with the following
prefixes:

COOP\_

GDOI\_

GM\_

KS\_

External References
===================

Cisco IOS GETVPN Start Page

http://www.cisco.com/go/getvpn

Cisco GETVPN Configuration Guide (15.1M&T) - Last Update 2/10/2011

http://www.cisco.com/c/en/us/td/docs/ios/sec_secure_connectivity/configuration/guide/convert/sec_get_vpn_15_1_book/sec_encrypt_trns_vpn.html


Cisco GETVPN G-IKEv2 configuration Guiode (IOS XE 3S) - Last Update 9/2/2017

http://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_conn_getvpn/configuration/xe-3s/sec-get-vpn-xe-3s-book/sec-get-vpn-gikev2.html
