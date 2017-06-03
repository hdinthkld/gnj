########################################################
Cisco IOS IKEv1 VPN with Dynamic VTI with Pre-shared Keys
########################################################

In this section we will configure a hub router that is able to offer VPN tunnels
to a unknown number of dynamic VPN peers

This is useful where you may need to rapidly deploy a varied number of sites
and do not want to have to reconfigure the hub router everytime a new site
is activated.

The configuration we will be putting in place is suitable when
the remote peers are using dynamic IP addressing but note that as we are
using pre-shared key authentication the same key will be accepted from any
IP address.  This is not as secure as it could be and in a production
deployment the use of digital signatures should be considerd.

It is assumed that the router already have basic IP connectivity and WAN
routing is in place.

After the IPSec tunnel is configured working we will also setup dynamic routing
through the tunnel.

Configuration Steps
===================

On all devices:
#. Configure a loopback to use a tunnel IP
#. Configure the Keyring
#. Configure the ISAKMP Policy
#. Configure the ISAKMP Profile
#. Configure the IPSec Proposal
#. Configure the IPSec Profile
#. Configure dynamic routing

On the Hub:
#. Configure the tunnel interface template

On the Spokes:
#. Configure a static tunnel interface



Step 1: Define the Loopback Interface
-------------------------------------

.. code-block:: none

  interface loopback <id>
    ip address <ip> 255.255.255.255

Step 1: Define the PSK Keyring
----------------------------------

For the hub it's necessary to define a wildcard PSK if the spokes will
be using dynamic IP addresses.  If they will be static they can be defined
individually but that would loose some of the flexibility of the solution.

.. code-block:: none

  crypto  keyring <keyring-name>
    ! note the use of a wildcard key, this is used by any connecting peer
    pre-shared-key address 0.0.0.0 key <psk>

On the spokes the PSK can be defined with just the Hubs IP:
.. code-block:: none

  crypto  keyring <keyring-name>
    pre-shared-key address <hub-ip> key <psk>

Step 1: Configure the ISAKMP Policy
----------------------------------

.. code-block:: none

  crypto isakmp policy <priority-number>
    authentication pre-shared
    encryption <encryption-algorithm>
    hash <integrity-algorithm>
    group <dh-group>
    lifetime <seconds>

Step 3: Configure the ISAKMP Profile
----------------------------------

.. code-block:: none

  crypto isakmp profile <isakmp-profile-name>
    match identity address 0.0.0.0
    keyring <keyring-name>
    virtual-template <template-id>

Step 4: Configure the IPSec Transform Set
-----------------------------------------

.. code-block:: none

  crypto ipsec transform-set <ts-name> <encryption-algorithm> <integrity-algorihm>
    mode tunnel

Step 5: Configure the IPSec Profile
------------------------------------

.. code-block:: none

  crypto ipsec profile <ipsec-profile-name>
    set transform-set <ts-name>
    set security-association lifetime seconds <seconds>
    set isakmp-profile <isakmp-profile-name>

Step 6: Define the tunnel interfaces
----------------------------------

On the Hub we will configure a template that will be cloned each time a
client connects.

.. code-block:: none

  interface virtual-template <template-id> type tunnel
    ip unnumbered loopback <id>
    tunnel mode ipsec ip
    tunnel destination dynamic
    tunnel source <wan-interface>
    tunnel protection ipsec profile <ipsec-profile-name>

On the Spokes we can configure a nubmer tunnel interface:

.. code-block:: none

  interface Tunnel <id>
    ip unnumbered loopback <id>
    tunnel mode ipsec ip
    tunnel destination <hub-ip>
    tunnel source <wan-interface>
    tunnel protection ipsec profile <ipsec-profile-name>

Complete Example
================

The Hub config could be performed as follows:

.. code-block:: none

  interface loopback 0
    ip address 10.0.0.1 255.255.255.255

  crypto  keyring DVTI-KEYRING
    pre-shared-key address 0.0.0.0 key mysecretkey

  crypto isakmp policy 10
    authentication pre-share
    encryption 3des
    hash md5
    group 2
    lifetime 86400

  crypto isakmp profile DVTI-ISAKMP-PROF
    match identity address 0.0.0.0
    keyring DVTI-KEYRING
    virtual-template 1

  crypto ipsec transform-set ESP-3DES-MD5 esp-3des esp-md5-hmac
    mode tunnel

  crypto ipsec profile IPSEC-PROF
    set transform-set ESP-3DES-MD5
    set security-association lifetime seconds 28800
    set isakmp-profile DVTI-ISAKMP-PROF

  interface virtual-template 1 type tunnel
    ip unnumbered  loopback0
    tunnel mode ipsec ipv4
    tunnel destination dynamic
    tunnel source FastEthernet0/0
    tunnel protection ipsec profile IPSEC-PROF

  router eigrp 10
    no auto-summary
    network 10.0.0.0 0.0.0.255
    network 10.1.0.0 0.0.255.255

The Spokes could then be configured as follows:

.. code-block:: none

  interface loopback 0
    ip address 10.0.0.2 255.255.255.255

  crypto isakmp policy 10
    authentication pre-share
    encryption 3des
    hash md5
    group 2
    lifetime 86400

  crypto keyring DVTI-KEYRING
    pre-shared-key address 192.168.1.1 key mysecretkey

  crypto isakmp profile DVTI-ISAKMP-PROF
    match identity address 192.168.1.1
    keyring DVTI-KEYRING

  crypto ipsec transform-set ESP-3DES-MD5 esp-3des esp-md5-hmac
    mode tunnel

  crypto ipsec profile IPSEC-PROF
    set transform-set ESP-3DES-MD5
    set security-association lifetime seconds 28800
    set isakmp-profile DVTI-ISAKMP-PROF

  interface tunnel 12
    ip unnumbered loopback 0
    tunnel mode ipsec ipv4
    tunnel source FastEthernet0/0
    tunnel destination 192.168.1.1
    tunnel protection ipsec profile IPSEC-PROF

  router eigrp 10
    no auto-summary
    network 10.0.0.0 0.0.0.255
    network 10.2.0.0 0.0.255.255


When ever a new spoke needs to be deployed the same config as above can be used,
just change the following:

#. Loopback IP
#. Subnets advertised by routing protocol
