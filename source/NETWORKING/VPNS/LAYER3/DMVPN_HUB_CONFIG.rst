#########################################
Dynamic Multipoint VPN Hub Configuration
#########################################

Single Hub with Pre-Shared Key Authentication
=============================================

.. rubric:: Summary of Steps

1. Pre-requisites
2. Define Phase 1 Policy
3. Define Phase 2 Policy
4. Define Authentication Credentials
5. Define IPSec Profile
6. Configure Multipoint GRE interface
7. Configure Dynamic Routing

.. rubric:: Step 1: Pre-requisites

* Ensure WAN/LAN intefaces are configured
* Ensure Routing is in place
* Verify initial connectivity is in place

.. rubric:: Step 2: Define Phase 1 Policy

::

  crypto isakmp policy <priority>
    authentication pre-share
    encryption <encryption-algorithm>
    group <dh-group>
    hash <integrity-algorithm>
    lifetime <seconds>

.. rubric:: Step 3: Define Authentication Credentials

::

  crypto isakmp key <psk> address <peer-ip-or-wildcard>


.. rubric:: Step 3: Define Phase 2 Policy

::

  crypto ipsec transform-set <ipsec-ts-name> <encryption-algorithm> <integrity-algorithm>
    mode transport

.. rubric:: Step 4: Define IPSec Profile

::

  crypto ipsec profile <ipsec-profile-name>
    set transform-set <ipsec-ts-name>

.. rubric:: Step 5a: Configure Dynamic Routing (EIGRP)

::

  router eigrp <as-number>
    no auto-summary
    network <dmvpn-subnet> <dmvpn-mask>
    network <lan-subnet> <lan-mask>

.. rubric:: Step 5b: Configure Multipoint GRE Interface

::

  interface tunnel <tunnel-id>
    ip address <dmvpn-subnet-ip> <dmvpn-mask>
    tunnel source <wan-interface>
    tunnel mode gre multipoint
    tunnel key <key-number>

    ! automatically create NHRP mapping for spokes
    ip nhrp map multicast dynamic

    ! Define the network password
    ip nhrp authentication <password>

    ! Set a unique network-id per DMVPN network
    ip nhrp network-id <net-id>

    ! Define how long to keep mapping if no updates are receives
    ip nhrp holdtime <seconds>

    tunnel protection ipsec profile shiva

    ! If using EIGRP Routing
    ! Ensure that the real advertising router is shown in routing table
    no ip next-hop-self eigrp <as>
    ! Allow routing updates to be sent out of the same interface as it
    ! received on.
    no ip split-horizon eigrp <as>

.. rubric:: Step 5b:: Configure Dynamic Routing (OSPF)

.. todo:: Complete example configruation for OSPF


Dual Hub with Pre-Shared Key Authentication
===========================================

Additional Steps
----------------

#. On Secondary Hub

  #. Define static mapping to primary hub (make the secondery hub an client to the primary hub)
  #. Define NHS server of primary hub
