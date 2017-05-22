##########################################
Dynamic Multipoint VPN Spoke Configuration
##########################################

Introduction
============

Configuration Steps
===================

Single Hub with Pre-Shared Key Authentication
=============================================

.. rubric:: Summary of Steps

1. Pre-requisites
2. Define Phase 1 Policy
3. Define Phase 2 Policy
4. Setup Authentication
5. Define IPSec Profile
6. Configure Multipoint GRE interface
7. Configure Dynamic Routing

.. rubric:: Step 1: Pre-requisites

* Ensure WAN/LAN intefaces are configured
* Ensure public WAN and LAN Routing is in place
* Verify initial connectivity is in place

.. rubric:: Step 2: Define Phase 1 Policy

::

  crypto isakmp policy <priority>
    authentication pre-share
    encryption <encryption-algorithm>
    hash <integrity-algorithm>
    group <dh-group>
    lifetime <seconds>

.. rubric:: Step 3: Define Phase 2 Policy

::

  crypto ipsec transform-set <ipsec-ts-name> <encryption-algorithm> <integrity-algorithm>
    mode transport

.. rubric:: Step 4: Define Authentication Credentials

::

  crypto isakmp key <psk> addresss <peer-ip-or-wildcard> [<wildcard-mask>]

.. rubric:: Step 5: Define IPSec Profile

::

  crypto ipsec profile <ipsec-profile-name>
  set transform-set <ipsec-ts-name>

.. rubric:: Step 6: Configure GRE Interface

::

  interface tunnel <tunnel-id>
    ip address <dmvpn-ip> <dmvpn-mask>

    tunnel source <wan-inteface>
    tunnel mode gre multipoint
    tunnel key <id>

    ! Define static mapping of hub private IP to it's public IP
    ip nhrp map <dmvpn-tunnel-ip> <nbma-public-ip>

    ! Send NHRP queries to the hub
    ip nhrp map multicast <nbma-public-ip>

    ip nhrp authentication <password>
    ip nhrp network-id <id>
    ip nhrp holdtime <seconds>

    ! Define the NHRP Server
    ip nhrp nhs <dmvpn-private-ip>

    tunnel protection ipsec profile <ipsec-profile-name>


    ! If using EIGRP routing
    no ip split-horizon eigrp <as>
    no ip next-hop-self eigrp <as>

    ! If using OSPF routing
    ! TODO

.. rubric:: Step 7a: Configure Dynamic Routing (EIGRP)

::

  router eigrp <as>
    no auto-summary
    network <dmvpn-tunnel-subnet> <dmvpn-mask>
    network <lan-subnet> <lan-mask>

.. rubric:: Step 7b: Configure Dynamic Routing (OSPF)

.. todo:: Document OSPF configuration


Dual Hub with Pre-Shared Key Authentication
===========================================

Additional Steps
----------------

#. Spokes should be configured with static mapping of both hubs
#. Spokes should be configured with both hubs as their NHS servers
