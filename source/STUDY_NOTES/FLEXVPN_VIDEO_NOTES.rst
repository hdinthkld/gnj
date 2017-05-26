##################
FlexVPN Video Notes
##################

Introduction
============

* IKEv2 & S2S = FlexVPN
* IKEv2 & RA = FlexVPN
* Can be used with legacy crypto maps and or VTI

* IKEv2 improvements over IKEv1

  * 4-6 Messages
  * Check peer existance via cookies
  * VOIP Support
  * Uses Suite B Cryptography

* IKEv2 Steps

  * IKESA_INIT (2 msg)
  * IKE_AUTH+CREATE_CHILD_SA (2 msgs)
  * IKE_CREATE_SECOND_CHILD_SA (optional 2 messages)


Configuration
=============

FlexVPN Site-To-Site with Pre-Shared Key
----------------------------------------

Summary of Steps
^^^^^^^^^^^^^^^^

The following steps should be completed on each peer that needs to have a VPN
tunnel established.

#. Define IKEv2 proposal
#. Define IKEv2 Policy
#. Define PSK/Keyring
#. Define IKEv2 Profile
#. Define IPSec Transform Set
#. Override IPSec Security Association lifetime (optional)
#. Define Interesting Traffic
#. Map the profiles and policies to the VPN peer
#. Apply mapping to appropriate interface

.. note:: If existing VPNs are in place it's possible some of this configuration
          may already exist.  Where possible a consistly high-strength security
          policy should be used to enable reuse of existing configuration.

Define the IKEv2 Proposal
^^^^^^^^^^^^^^^^^^^^^^^^^

::

  crypto ikev2 proposal <id>
    encryption <encryption>
    integrity <hash>
    group <dh-group>


Define the IKEv2 Policy
^^^^^^^^^^^^^^^^^^^^^^^^^

::

 crypto ikev2 policy <id>
  proposal <id>


Define the IKEv2 Keyring or PSK
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. rubric:: Using a Keyring

::

 crypto ikev2 keyring <name>
   peer <name>
     address <ip>
     pre-shared-key <psk>

.. rubric:: And for preshared key

::

  crypto ikev2 key <psk> address <ip>

.. todo:: Check if this is even valid?

::

  crypto ikev2 profile <name>
    match identity remote address
    authentication local pre-share
    authentication remote pre-share
    keyring <name>

::

  crypto ipsec transform-set <name> <encryption-algorithm> <integrity-algorithm>

Override the default IPSec SA lifetime (optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: This can also be set for individual peers by defining in the later
          crypto map.

::

  crypto ipsec security-association lifetime seconds <seconds>


Define the traffic to be encrypted over the VPN tunnel
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  access-list <name-or-id> permit ip <local-net> <local-mask> <remote-net> <remote-mask>


Map the encryption settings to the appropriate VPN peer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  crypto map <name> <seq-id> ipsec-isakmp
    set transform-set <name>
    set peer <ip>
    match address <name-or-id>
    set ikev2-profile <id>

Apple the encryption mapping to the appropriate interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  interface <type><slot/num>
    crypto map <name>


FlexVPN S2S with RSA
--------------------

.. warning:: It is important that all devices have a consistent time (preferably)
          via NTP.  If not authentication failures may occur due to devices
          thinking that certificates are either not yet valid or have expired.

Summary of Steps
^^^^^^^^^^^^^^^^

#. Define the trusted CA
#. Authenticate the CA
#. Enroll with the CA
#. Define IKEv2 Profile with RSA Authentication
#. Repeat all other steps as with PKI

Define the trusted CA
^^^^^^^^^^^^^^^^^^^^^

::

  crypto pki trustpoint <ca-name>
    enrollment url <url>
    revocation-check none

Authenticate the CA
^^^^^^^^^^^^^^^^^^^

::

  crypto pki authenticate <ca-name>

Enroll the device with the new CA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  crypto pki enroll <ca-name>

Define the IKEv2 Profile
^^^^^^^^^^^^^^^^^^^^^^^^^

::

  crypto ikev2 profile <name>
    match identity remote address <ip>
    authentication local rsa-sig
    authentication remote rsa-sig
    pki trustpoint <ca-name>

Define the tunnel interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  interface tunnel <id>
    ip address <ip> <mask>
    tunnel mode ipsec ipv4
    tunnel protection ipsec profile <ipsec-profile-name>


FlexVPN Remote Access with PSK
------------------------------

Summary of Steps
^^^^^^^^^^^^^^^^

.. rubric:: Shared Configuration

#. Define the IKEv2 Keyring (Use wildcard IP if clients have dynamic IP)
#. Define the IKEv2 Policy
#. Define the IPSec Proposal
#. Define the split-tunnel ACL
#. Define Authorisation Policy
#. Define the  IKEv2 Profile
#. Define the IPSec Transform Set
#. Override the default IPSec SA Lifetime (optional)

.. rubric:: For Remote Access Server


#. Define IP Pool
#. Define virtual tempalte

.. rubric:: For Remote Access Client

#. Define tunnel interface
#. Define IKEv2 client

Remote Access Server/Hub Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  crypto ikev2 authorization policy
    pool <ip-pool-name>
    route set access-list <split-tunnel-acl>

::

  int virtual-template <id> type tunnel
    ip unnumbered <int>
    tunnel source <int>
    tunnel protection ipsec profile <ipsec-profile-name>

::

  crypto ikev2 profile <id>
    virtual-template <id>
    aaa authorization group psk <keyring-name>
    aaa authorization group psk list default <ikev2-auth-pol-name>
    pki trustpoint <ca-name>


Remote Access Client Steps
^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  crypto ikev2 keyring
    peer <name>
      addresss <ip-or-wildcard> <mask>
      pre-shared-key <psk>

::

  interface tunnel <id>
   tunnel source <int>
   tunnel destination <dynamic>
   tunnel mode ipsec ipv4
   tunnel protection ipsec profile <ipsec-profile-name>

::

  crypto ikev2 client flexvpn <name>
    client connect tunnel <id>
    peer <seq> <ip>


FlexVPN Remote Access with RSA
------------------------------

.. rubric:: On Both Hub and Remote Clients

#. Setup trusted CA
#. Authenticate the CA
#. Enroll with the CA
#. Define the IKEv2 Authorization Policy
#. Complete remaining steps as per basic setup

.. rubric:: On Hub

#. Define the remote client connection template

.. rubric:: On Remote Clients

#. Define the VPN tunnel interface
#. Define how to connect to the hub


FlexVPN Site-To-Site with IPv6 and PSK
--------------------------------------

Setup is practically the same as with IPv4, with the following changes

#. Use 'ipv6 address' on interface addressing
#. All peer addressing should use IPv6 addresses (if used on public network)
#. Tunnel mode should be specified as 'ipsec ipv6'

.. todo:: Is it possible to do an IPv4 public IP but then do IPv6 private subnets?
