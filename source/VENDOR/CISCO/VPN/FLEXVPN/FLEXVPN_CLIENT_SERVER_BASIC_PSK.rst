###############################################
Cisco FlexVPN Basic Client/Server Configuration
###############################################

Overview
========

This configuration will demonstrate the absolute minimum configuration that
is required in order to get a FlexVPN spoke acting as a client to establish
a vpn tunnel to a FlexVPN hub acting as the server.


FlexVPN Server Configuration
============================

The following elements must be configured on the Hub acting as the FlexVPN
server:

#. Enable AAA
#. Define the local subnets to be encrypted
#. Create Address Pool
#. IKEv2 Keyring
#. IKEv2 Authorisation Policy
#. IKEv2 Profile
#. IPSec Profile
#. Tunnel interface template

It is assumed that basic LAN/WAN connectivity and routing is already in
place and tested.

Enable AAA
----------

In case we need to configure multiple differnet authorisation policies we
won't change the default authorisation list, instead creating one unique for
this deployment.

This for example would allow one method to use local PSKs and another
using RADIUS if so needed.

.. code-block:: none

  aaa new-model
  aaa authorization network <aaa-method-list-name> local

Define the local subnets
------------------------

.. code-block:: none

  ip access-list standard <crypto-acl-name>
    permit <local-subnet> <local-mask>

Create the Address Pool
-----------------------

.. code-block:: none

  ip local pool <ip-pool-name> <start-ip> <end-ip>

Configure the IKEv2 Keyring
---------------------------

For this example we will be using symmetric pre-shared keys but it is also
possible to use assymetric by specifying different 'local' and 'remote' values.

.. code-block:: none

  crypto ikev2 keyring <ikev2-keyring-name>
    peer <peer-name-or-any>
     address <ip-or-wildcard>
       pre-shared-key <psk>

Configure the IKEv2 Authorisation policy
----------------------------------------

The authorisation policy specifies the attributes that will apply to clients
who are successfully authorised against this policy.

.. code-block:: none

  crypto ikev2 authorization policy <ikev2-auth-policy-name>
    ! Define the pool used to give clients an IP address
    pool <ip-pool-name>

    ! Specify the local subnets that will be advertised to clients (split-tunnel)
    route set access-list <crypto-acl-name>

Define the IKEv2 Profile
------------------------

.. code-block:: none

  crypto ikev2 profile <ikev2-profile-name>
    match identity remote address <client-ip-or-wildcard> <optional-mask>
    authentication remote pre-share
    authentication local pre-share
    keyring local <ikev2-keyring-name>
    aaa authorization group psk list default <ikev2-auth-policy-name>
    virtual-template <tunnel-interface-number>

Define the IKEv2 Profile
------------------------

.. code-block:: none

  crypto ipsec profile <ipsec-profile-name>
    set ikev2-profile <ikev2-profile-name>


Define the interface template
-----------------------------

.. code-block:: none

  interface virtual-template <tunnel-interface-number>
    ip unnumbered <wan-interface>
    tunnel source <wan-interface>
    tunnel mode ipsec ipv4
    tunnel protection ipsec profile <ipsec-profile-name>


FlexVPN Client Configuration
============================

We will configure the following elements:

#. Enable AAA
#. Define the local subnets to be encrypted
#. IKEv2 Keyring
#. IKEv2 Authorisation Policy
#. IKEv2 Profile
#. IPSec Profile
#. Tunnel Interface
#. FlexVPN Client Profile

Enable AAA
----------

Unlike the server we are less likely to need multiple policies on the client
so in this instance we just change the defalt one.

.. code-block:: none

  aaa new-model
  aaa authorization network default local

Define the local subnets to be encrypted
----------------------------------------

.. code-block:: none

  ip access-list standard <crypto-acl-name>
    permit <local-subnet> <local-mask>


Create the IKEv2 Keyring
------------------------

.. code-block:: none

  crypto ikev2 keyring <ikev2-keyring-name>
    peer <hub-name>
      address <hub-ip>
        pre-shared-key psk

Create the IKEv2 Authorization Policy
-------------------------------------



.. code-block:: none

   crypto ikev2 authorization policy <ikev2-auth-policy-name>
     route set access-list <crypto-acl-name>
     route set interface

Create the IKEv2 Profile
------------------------

Same config as the server but instead using the default authorization
method list.

.. code-block:: none

   crypto ikev2 profile <ikev2-profile-name>
     match identity remote address <hub-ip>
     authentication remote pre-share
     authentication local pre-share
     keyring local <ikev2-keyring-name>
     aaa authorization group psk list default <ikev2-auth-policy-name>

Create the IPSec Profile
------------------------

.. code-block:: none

   crypto ipsec profile <ipsec-profile-name>
     set ikev2-profile <ikev2-profile-name>


Create the tunnel interface
---------------------------

.. code-block:: none

  interface tunnel <tunnel-interface-number>
    tunnel mode ipsec ipv4
    ip address negotiated
    tunnel source <wan-interface>
    tunnel destination dynamic
    tunnel protection ipsec profile <ipsec-profile-name>


Create the FlexVPN Client Profile
---------------------------------

.. code-block:: none

  crypto ikev2 client flexvpn <flexvpn-profile-name>
    peer <seq-no> <hub-ip>
    client connect tunnel <tunnel-interface-number>
