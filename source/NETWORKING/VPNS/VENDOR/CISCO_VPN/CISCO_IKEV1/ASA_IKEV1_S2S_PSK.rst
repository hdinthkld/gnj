.. _asa_ikev1_s2s_psk:

##############################################################
Cisco ASA IKEv1 VPN Configuration with Pre-Shared Keys Example
##############################################################

Introduction
============

In this example we'll configure a Cisco ASA to talk with a remote peer
using IKEv1 with symmetric pre-shared keys.

Configuration Steps
===================

#. Define the Encryption Domain
#. Specify the Phase 1 Policy
#. Specify the Phase 2 Proposal
#. Define the connection profile
#. Configure the Crypto Map
#. Bind the Crypto Map to the appropriate interface
#. Enable IKEv1 on the appropriate interface

Define the Encryption Domain
============================

The encryption domain specifies traffic that should be encapsulated within
IPSec prior to leaving the external interface.  Any traffic not matching
this ACL will be sent out the interface in plain text, assuming it does
not match any other configured VPNs.

The ACL should also take into account of any NAT'ing that has taken place as
the encryption occurs post-NAT.

.. code-block:: none

  access-list <encryption-acl-name> permit <local-net> <local-mask> <remote-net> <remote-mask>

Specify the Phase 1 Policy
==========================

.. code-block:: none

  crypto ikv1 policy <seq-number>
    encryption <encryption-algorithn>
    hash <integrity-algorithm>
    group <dh-group>
    lifetime <seconds>
    authentication pre-share

Specify the Phase 2 Proposal
============================

In newer releases on Tunnel mode is support, Transport has been removed

.. code-block:: none

  crypto ipsec ikev1 transform-set <ipsec-proposal-name> <encryption-algorithm> <integrity-algorithm>


Define the connection profile
=============================

In general speak a connection profile defines the properties of how the VPN
will run and what access will be permitted.  It is called as such in the ASDM
but through the CLI we need to configure a tunnel-group

In earlier versions of the ASA code (pre-8.0) the preshared key was
configured globally as with earlier IOS versions.

.. code-block:: none

  tunnel-group <vpn-peer-ip> ipsec-l2l
  tunnel-group <vpn-peer-ip> ipsec-attributes
    ikev1 pre-shared-key <psk>
  tunnel-group <vpn-peer-ip> general-attributes
  ! Define additional settings such as default group policy



Configure the Crypto Map
========================

Only a single Crypto Map can be bound to an interface, so if one already exists
use a different sequence number to existing entries.

If remote access is configured it is important to ensure that site-to-site
entries have a lower sequence nubmer than the remote access one.

.. code-block:: none

   crypto map <cm-name> <seq-number> match address <encryption-acl-name>
   crypto map <cm-name> <seq-number> set peer <peer-ip>
   crypto map <cm-name> <seq-number> set transform-set <ipsec-proposal-name>
   crypto map <cm-name> <seq-number> set security-association lifetime seconds <seconds>


Bind the Crypto Map to the interface
====================================

If this is the first VPN (either IKEv1 or IKEv2) being setup, it will be
necessary to bind the Crypto Map to the interface facing the remote peer(s).
Otherwise this will already have been configured.

.. code-block:: none

  crypto map <cm-name> interface <ifname>

Enable IKEv1 on the the interface
==================================

If this is the first IKEv1 VPN being setup, it will be necessary to bind the
Crypto Map to the interface facing the remote peer(s).  Otherwise this will
already have been configured.

.. code-block:: none

  crypto ikev1 enable <ifname>
