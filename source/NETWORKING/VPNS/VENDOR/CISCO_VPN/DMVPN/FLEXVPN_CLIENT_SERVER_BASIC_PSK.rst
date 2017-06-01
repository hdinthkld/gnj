###############################################
Cisco FlexVPN Basic Client/Server Configuration
###############################################

Overview
========

This configuration will demonstrate the absolute minimum configuration that
is required in order to get a FlexVPN spoke acting as a client to establish
a vpn tunnel to a FlexVPN hub acting as the server.

Whilst this will be a minimal config, all elements will be configured and
none of the default policies will be used.  This is illustrate all required
components and offers protection in case someone (e.g. Cisco) were to
change the defaults depending on the version of IOS used.

FlexVPN Server Configuration
============================

The following elements must be configured on the Hub acting as the FlexVPN
server:

#. IKEv2 Policy (optional)
#. IKEv2 Keyring
#. IKEv2 Proposal
#. IKEv2 Profile
#. IKEv2 Name Mangler
#. IKEv2 Authorisation Policy
#. IPSEC Transform (optional)
#. IPSec Profile
#. Tunnel interface template

It is assumed the basic LAN/WAN connectivity and routing is already in
place and tested.

Configure the Network AAA List, using local database
-----------------------------------------------------

.. code-block:: none

  aaa authorixzation network <aaa-authz-list> local

Configure the AAA attribute list
--------------------------------

.. code-block:: none

  aaa attribute list <attribute-list-name>
    attribute type interface-config "tunnel key 0"


Configure IKEv2 Keyring
-----------------------

FlexVPN is based on IKEv2 so we are able to use either a symmetric or
asymmetric preshared key.  For simplicity we'll just use a symmetric key.

.. code-block:: none

  crypto ikev2 keyring <ikev2-keyring-name>
    peer <spoke-hostname-or-wildcard>
      address <spoke-ip-or-wildcard>
      pre-shared-key <psk>

Configure IKEv2 Proposal
------------------------

.. code-block:: none

  crypto ikev2 proposal <ikev2-proposal-name>
    encryption <encryption>
    integrity <integrity>
    group <dh-group>


Configure the IKEv2 Name Mangler
--------------------------------

.. code-block:: none

  crypto ikev2 name-mangler <ikev2-mangler-name>
    fqdn domain

Configure the IP Address Pool
-----------------------------

.. code-block:: none

  ip local pool <ip-pool-name> <start-ip> <end-ip>


Define the subnets reachable over the VPN by clients
----------------------------------------------------

.. code-block:: none

  ip access-list standard <crypto-acl-name>
    ! As many entries as necessary
    permit <local-subnet> <local-mask>


Configure the IKEv2 Authorisation Policy
----------------------------------------

.. code-block:: none

  crypto ikev2 authorization policy <ikev2-auth-policy-name>
    aaa attribute list <aaa-list-name>

    ! Configure default domain
    def-domain <local-domain>

    ! Specify DNS Servers
    dns <server1-ip> <server2-ip>

    ! Specify address pool for giving IPs to clients
    pool <ip-pool-name>

    ! Define the subnets reachable over the VPN
    route set access-list <crypto-acl-name>



Configure the IPSec Transform
-----------------------------

.. code-block:: none

  crypto ipsec transform-set <ipsec-ts-name> <encryption> <integrity>
    mode tunnel


Configure the IKEv2 Profile
---------------------------

.. code-block:: none

  crypto ikev2 profile <ikev2-profile-name>
    match identity remote fqdn <spoke-fqdn>
    identity local fqdn <local-fqdn>
    authentication local pre-share
    authentication remote pre-share
    keyring local <ikev2-keyring-name>
    aaa authentication local pre-share <psk>
    aaa authorization group psk list <aaa-authz-list>
    config-exchange set accept
    virtual-template <virtual-template-number>

Configure the IPSec Profile
---------------------------

.. code-block:: none

  crypto ipsec profile <ipsec-profile-name>
    set transform-set <ipsec-ts-name>
    set ikev2-profile <ikev2-profile-name>
    ! Optional
    set security-association lifetime seconds <seconds>

Configure Tunnel Interface Template
-----------------------------------

.. code-block:: none

   interface virtual-template<id> type tunnel
     shutdown
     ip unnumbered <loopback-interface-number>
     tunnel mode ipsec ipv4
     tunnel protection ipsec profile <ipsec-profile-name>



FlexVPN Client Configuration
============================

We will configure the following elements:

#. IKEv2 Keyring
#. IKEv2 Proposal
#. IKEv2 Profile
#. IPSec Transform
#. IPSec Profile
#. Tunnel Interface
#. FlexVPN Client Profile

Configure IKEv2 Keyring
-----------------------

FlexVPN is based on IKEv2 so we are able to use either a symmetric or
asymmetric preshared key.  For simplicity we'll just use a symmetric key.

.. code-block:: none

  crypto ikev2 keyring <ikev2-keyring-name>
    peer <hub-hostname>
      address <hub-ip>
      pre-shared-key <psk>

Configure the Encryption Domain
-------------------------------

.. code-block:: none

  ip access-list standard <crypto-acl-name>
    ! Repeat for as many lines as necessary
    permit ip <local-subnet> <local-mask>


Configure the IKEv2 Authorization Policy
----------------------------------------

.. code-block:: none

  crypto ikev2 authorization policy <auth-policy-name>
    ! replaced subnet-acl
    route set access-list <crypto-acl-name>
    route set interface
    router accept any

Configure IKEv2 Proposal
------------------------

.. code-block:: none

  crypto ikev2 proposal <ikev2-proposal-name>
    encryption <encryption>
    integrity <integrity>
    group <dh-group>

Configure the IKEv2 Profile
---------------------------

The IKEv2 profile is used to determine what identities to use for each peer
and how they will authenticate. In this instance the full FQDN hostname
will be used to match identities and pre-shared-key will be used for
authentication, using the keyring configured above.

.. code-block:: none

  crypto ikev2 profile <ikev2-profile-name>
    match identity remote fqdn <hub-fqdn>
    identity local fqdn <local-fqdn>
    authentication local pre-share
    authentication remote pre-share
    keyring local <ikev2-keyring-name>
    aaa authorization group psk list <auth-policy-name>
    config-mode set

Configure the IPSec Transform
-----------------------------

.. code-block:: none

  crypto ipsec transform-set <ipsec-ts-name> <encryption> <integrity>
    mode tunnel


Configure the IPSec Profile
---------------------------

.. code-block:: none

crypto ipsec profile <ipsec-profile-name>
  set transform-set <ipsec-ts-name>
  set ikev2-profile <ikev2-profile-name>

  ! Optional
  set security-association lifetime seconds <seconds>

Configure the tunnel interface
------------------------------

.. code-block:: none

  interface Tunnel <tunnel-number>
    ip address negotiated
    tunnel source <wan-interface>
    tunnel destination dynamic
    tunnel protection ipsec profile


Configure FlexVPN Client Profile
--------------------------------

.. code-block:: none

  crypto ikev2 client flexvpn <flexvpn-client-profile-name>
    peer <seq-number> <hub-ip>
    connect {manual | auto }
    client connect Tunnel <tunnel-number>
