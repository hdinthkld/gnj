##########################################################
Cisco Dynamic Multipoint VPN with PSK Basic Configuration
##########################################################

In this section 3 routers will be configured to provide a basic DMVPN. One
of the routers will act as a hub, the remaining two as the spokes.


Hub Configuration Steps
=======================

Step 1: Define the IKE Phase 1 Policy
-------------------------------------

.. code-block:: none

  crypto isakmp policy <id>
    encryption <encryption-algorithm>
    hashing <integrity-algorithm>
    authentication pre-share
    group <dh-group>
    lifetime <seconds>

Step 2: Define the Pre-Shared Key
---------------------------------

.. code-block:: none

  ! Same key used for all spokes regardless of IP Address
  crypto isakmp key <psk> address 0.0.0.0


Step 3: Define the IPSec Proposal
---------------------------------

.. code-block:: none

  crypto ipsec transform-set <ts-name> <encryption-algorithm> <integrity-algorithm>
    mode { tunnel | transport }

Step 4: Define the IPsec Profile
--------------------------------

.. code-block:: none

  crypto ipsec profile <ipsec-profile-name>
    set transform-set <ts-name>
    set security-association lifetime seconds <seconds>

Step 5: Create the tunnel interface
-----------------------------------

.. code-block:: none

  interface tunnel <id>
    ip address <private-ip> <private-mask>
    tunnel mode gre multipoint

    tunnel key <unique-number>
    tunnel source <wan-interface>

    ip nhrp nhs map multicast dynamic
    ip nhrp authentication <nhrp-password>
    ip nhrp network-id <network-id>
    ip nhrp holdtime <seconds>

    tunnel protection ipsec profile <ipsec-profile-name>

    no shutdown

Spoke Configuration Steps
=========================

Step 1: Define the IKE Phase 1 Policy
-------------------------------------

.. code-block:: none

  crypto isakmp policy <id>
    encryption <encryption-algorithm>
    hashing <integrity-algorithm>
    authentication pre-share
    group <dh-group>
    lifetime <seconds>

Step 2: Define the Pre-Shared Key
---------------------------------

.. code-block:: none

  ! Same key used for all spokes regardless of IP Address
  crypto isakmp key <psk> address 0.0.0.0


Step 3: Define the IPSec Proposal
---------------------------------

.. code-block:: none

  crypto ipsec transform-set <ts-name> <encryption-algorithm> <integrity-algorithm>
    mode { tunnel | transport }

Step 4: Define the IPsec Profile
--------------------------------

.. code-block:: none

  crypto ipsec profile <ipsec-profile-name>
    set transform-set <ts-name>
    set security-association lifetime seconds <seconds>

Step 5: Define the Tunnel Interface
-----------------------------------

.. code-block:: none

  inteface tunnel <id>
    ip address <private-ip> <private-mask>
    tunnel mode gre multipoint
    tunnel key <unique-key-per-dmvpn>

    ip nhrp map <hub-dmvpn-private-ip> <hub-nbma-public-ip>
    ip nhrp nhs <hub-dmvpn-private-ip>
    ip nhrp map multicast <hub-nbma-public-ip>

    ip nhrp authentication <nhrp-password>
    ip nhrp network-id <unique-id-per-dmvpn>
    ip nhrp holdtime <seconds>

Routing Protocol Considerations
===============================

DMVPN can work with either Link State or Distance Vector protocols. However
considerations need to be made for each.

Distance Vector Protocols
-------------------------

In order to use routing protocols such as EIGRP and RIP, it necessary to
disable split horizon so that routing advertisements from the spokes can
be readvertised out of the single hubs interface.

In addition the routing updates should still contain the original peer that
advertised them, not the hops.  To achieve this Next Hop Self should be
disabled.

In the case of EIGRP these can be configured on the tunnel interface as follows:

.. code-block:: none

  interface tunnel <id>
    no split-horizon eigrp <as-number>
    no next-hop-self eigrp <as-number>

Link State Protocols
--------------------

Routing protocols such as OSPF will automatically ensure all peers receiving the
routing updates because this is the Designated Routers (DR) responsibility. It
however important to ensure that none of the spokes can become the DR.

It is also vital that the tunnel interface is set to us the network type of
"broadcast" to ensure that the DR/BDR election occurs.  If this is not set
and more than two router Ids are seen on the same subnet, this could result
in flapping neighbour relationships.

The above can be achieved by setting the priority of the spokes to 0 and
manually setting the network type on the tunnel as shown below:

.. code-block:: none

  interface tunnel <id>
    ip ospf network broadcast
    ip ospf priority 0


When using dual-hub, its important that the priority of the primary hub is
higher than that of the secondary. In the case of a primary hub failure, the
spokes will notice for themselves when the hold time has expired and
automatically start queying the secondary NHS.
