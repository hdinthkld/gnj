.. _asa_ikev2_s2s_psk:

#########################################################################
Cisco ASA IKEv2 VPN Configuration with Assymetric Pre-Shared Keys Example
#########################################################################

Introduction
============

In this example we'll configure a Cisco ASA to talk with a remote peer
using IKEv2 with assymetric pre-shared keys.

Configuration Steps
===================

#. Define the Encryption Domain
#. Specify the Phase 1 Policy
#. Specify the Phase 2 Proposal
#. Define the connection profile
#. Configure the Crypto Map
#. Bind the Crypto Map to the appropriate interface

Define the encryption domain
============================

.. code-block:: none

  access-list <encryption-domain-acl> <local-net> <local-mask> <remote-net> <remote-mask>


Define the Phase 1 Policy
=========================

The ASDM location for these settings is:

:menuselection:`Configure --> Site-To-Site VPN --> Advanced --> IKE Policies`

.. code-block:: none

  crypto ikev2 policy <seq-number>
    encryption <encryption-algorithm>
    integrity <integrity-algorithm>
    group <dh-group>
    lifetime <seconds>

Define the Phase 2 Proposal
===========================

The ASDM location for these settings is:

Configure > Site-To-Site VPN-> Advanced > IPSec Proposals (Transform Sets)

.. code-block:: none

  crypto ipsec ikev2 ipsec-proposal <ipsec-proposal-name>
    protocol esp encryption <encryption-algorithm>
    protocol esp integrity <integrity-algorithm>

Define the connection profile
=============================

The ASDM location for these settings is:

:menuselect:`Configure --> Site-To-Site VPN --> Advanced --> Tunnel Groups`

.. code-block:: none

  tunnel-group <peer-ip> type ipsec-l2l
  tunnel-group <peer-ip> ipsec-attributes
    ikev2 local-authentciation pre-shared-key <local-psk>
    ikev2 remote-authentcation pre-shared-key <remote-psk>
  tunnel-group <peer-ip> general-attributes
    ! Define other properties, such as default group policy

Define the crypto map
=====================

when creating the connection profiles in ASDM, part of the configuration
includes the interface and protocols to be allowed.  The selection can
be seen on the connection profiles section of ASDM:

:menuselection:`Configure --> Site-To-Site VPN --> Advanced --> Crypto Maps`

.. code-block:: none

  crypto map <cm-name> <seq-number> match address <encryption-domain-acl>
  crypto map <cm-name> <seq-number> set peer <peer-ip>
  crypto map <cm-name> <seq-number> set ikev2 transform-set <ipsec-proposal-name>
  crypto map <cm-name> <seq-number> set security-association lifetime seconds <seconds>

Bind the Crypto Map to the interface
====================================

If this is the first VPN (either IKEv1 or IKEv2) being setup, it will be
necessary to bind the Crypto Map to the interface facing the remote peer(s).
Otherwise this will already have been configured.

In ASDM as soon as any VPN is configured it will automatically bind a crypto
map to the selected interface. The binding can be seen at the following
location:

Configure > Site-To-Site VPN > Connection Profiles

.. code-block:: none

  crypto map <cm-name> interface <ifname>

Enable IKEv1 on the the interface
==================================

If this is the first IKEv2 VPN being setup, it will be necessary to bind the
Crypto Map to the interface facing the remote peer(s).  Otherwise this will
already have been configured.

In ASDM the selection of which protocol is enabled per-interface, can be seen
on the connection profiles section:

Configure > Site-To-Site VPN > Connection Profiles

.. code-block:: none

  crypto ikev2 enable <ifname>
