########################################################
Cisco IOS IKEv1 VPN with Static VTI with Pre-shared Keys
########################################################

In this section we will configure a pair of routers to communicate over a
statically configured VTI using GRE over IPSec.

This is useful in situations where you need to carry non-IP traffic through
IPSEC.

It is assumed that the router already have basic IP connectivity and WAN
routing is in place.

After the IPSec tunnel is configured working we will also setup dynamic routing
through the tunnel.

Configuration Steps
===================

#. Configure the PSK Keyring
#. Configure the ISAKMP Policy
#. Configure the ISAKMP Profile
#. Configure the IPSec Proposal
#. Configure the IPSec Profile
#. Configure the VTI tunnel using GRE over IPSec encapsulation
#. Configure routing via the tunnel

Step 1: Define the PSK Keyring
----------------------------------

.. code-block:: none

  crypto  keyring <keyring-name>
    pre-shared-key address <ip> key <psk>


Step 1: Confifigure the ISAKMP Policy
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
   match identity address <ip>
   keyring <keyring-name>

Step 4: Configure the IPSec Transform Set
-----------------------------------------

.. code-block:: none

  crypto ipsec transform-set <ts-name> <encryption-algorithm> <integrity-algorihm>
    mode transport

Step 5: Configure the IPSec Profile
------------------------------------

.. code-block:: none

  crypto ipsec profile <ipsec-profile-name>
    set transform-set <ts-name>
    set security-association lifetime seconds <seconds>
    set isakmp-profile <isakmp-profile-name>

Step 6: Configure the VTI interface
-----------------------------------

.. code-block:: none

  interface Tunnel <id>
    tunnel mode gre ip
    tunnel source <wan-interface>
    tunnel destination <remote-peer-ip>
    tunnel protection profile ipsec <ipsec-profile-name>
    ip address <ip> <mask>
    no shutdown


Step 6a: Configure routing (EIGRP)
--------------------------------

.. code-block:: none

  router eigrp <as-number>
    no auto-summary
    network <tunnel-subnet> <tunnel-mask>
    nework <lan-subnet> <lan-mask>

Step 6a: Configure routing (EIGRP)
--------------------------------

.. code-block:: none

  router ospf <process-id>

  interface tunnel <id>
    ip ospf  <process-id> area <area-id>
    ip ospf network point-to-point

Complete Example
================

The hub could be configured as follows:

.. code-block:: none

  crypto keyring VTI-KEYRING
    pre-shared-key address 192.168.2.2 key mysecretkey

  crypto isakmp policy 10
    authentication pre-share
    encryption 3des
    hash md5
    group 2
    lifetime 86400

  crypto isakmp profile VTI-ISAKMP-PROF
    match identity address 192.168.2.2
    keyring VTI-KEYRING

  crypto ipsec transform-set ESP-3DES-MD5 esp-3des esp-md5-hmac
    mode transport

  crypto ipsec profile VTI-IPSEC-PROF
    set transform-set ESP-3DES-MD5
    set security-association lifetime seconds 28800
    set isakmp-profile VTI-ISAKMP-PROF
    set pfs group2

  interface Tunnel 12
    tunnel mode gre ip
    tunnel source FastEthernet0/0
    tunnel destination 192.168.2.2
    tunnel protection ipsec profile VTI-IPSEC-PROF
    ip address 10.255.12.1 255.255.255.0
    no shutdown

  router eigrp 10
    no auto-summary
    network 10.255.12.0 0.0.0.255
    network 10.1.0.0 0.0.255.255


The spoke could be configured as follows

.. code-block:: none

  crypto keyring VTI-KEYRING
    pre-shared-key address 192.168.1.1 key mysecretkey

  crypto isakmp policy 10
    authentication pre-share
    encryption 3des
    hash md5
    group 2
    lifetime 86400

  crypto isakmp profile VTI-ISAKMP-PROF
    match identity address 192.168.1.1
    keyring VTI-KEYRING

  crypto ipsec transform-set ESP-3DES-MD5 esp-3des esp-md5-hmac
    mode transport

  crypto ipsec profile VTI-IPSEC-PROF
    set transform-set ESP-3DES-MD5
    set security-association lifetime seconds 28800
    set isakmp-profile VTI-ISAKMP-PROF
    set pfs group2

  interface Tunnel 12
    tunnel mode gre ip
    tunnel source FastEthernet0/0
    tunnel destination 192.168.1.1
    tunnel protection ipsec profile VTI-IPSEC-PROF
    ip address 10.255.12.2 255.255.255.0
    no shutdown

  router eigrp 10
    no auto-summary
    network 10.255.12.0 0.0.0.255
    network 10.2.0.0 0.0.255.255
