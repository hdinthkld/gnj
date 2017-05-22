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

    ! Enable DMVPN Phase 3
    ip nhrp redirect

.. rubric:: Step 5b: Configure Dynamic Routing (OSPF)

! If using OSPF routing
! Define the DMVPN network as a broadcast network type
ip ospf network broadcast
! Set the Primary Hub to have the highest OSPF Priority
ip ospf priority <priority>
! All the GRE tunnel interface to participate in OSPF
ip ospf <process-id> area <area-id>

.. todo:: Complete example configuration for OSPF
    router ospf <process-id>



Dual Hub with Pre-Shared Key Authentication
===========================================

Additional Steps
----------------

.. rubric:: Configure Secondary Hub

#. Same Configuration as Primary Hub
#. Define static mapping to primary hub (make the secondery hub an client to the primary hub)
#. Define NHS server of primary hub


Using DMVPN with Key Ring and DMVPN Profile
===========================================

Complete steps as per the previous configuration with the following differences:

.. rubric:: Step 1: Define the PSKs

::
  crypto keyring <kr-name>
    pre-shared-key addresss <ip> key <psk>

.. rubric:: Step 2: Define the ISAKMP Profile

::
  crypto isakmp profile <isakmp-profile-name>
    match identity address <ip-or-wildcard>
    keyring <kr-name>


.. rubric:: Step 3: Define the IPSEC Profile

::
  crypto ipsec profile <ipsec-profile-name>
    set transform-set <ts-name>
    set isakmp-profile <isakmp-profile-name>


Single Hub with RSA Authentication
==================================

.. rubric:: Step 1: Configure the trusted CA

::

  crypto ca trustpoint <ca-name>
    enrollment url <url>


.. rubric:: Step 2: Authenticate the CA Server

::
  crypto ca authenticate <ca-name>

.. rubric:: Step 3: Enroll with the CA Server

::
  crypto ca enroll <ca-name>


.. rubric:: Step 3: Define the Phase 1 Policy

::
  crypto isakmp policy <priority>
    authentication rsa-sig
    encryption
    hash
    group
    lifetime

Remaining steps are the same as with Pre-Shared Key Authentication
