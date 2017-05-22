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
    ! Define the DMVPN network as a broadcast network type
    ip ospf network broadcast
    ip ospf priority 0

    ! Enable DMVPN Phase 3
    ip nhrp shortcut

.. rubric:: Step 7a: Configure Dynamic Routing (EIGRP)


::

  router eigrp <as>
    no auto-summary
    network <dmvpn-tunnel-subnet> <dmvpn-mask>
    network <lan-subnet> <lan-mask>

.. rubric:: Step 7b: Configure Dynamic Routing (OSPF)

.. caution:: When using OSPF spoke devices should not become the DR or BDR,
          therefore set their priority to 0 so they do not take part in the
          election process


.. todo:: Document OSPF configuration


Dual Hub with Pre-Shared Key Authentication
===========================================

Additional Steps
----------------

#. Spokes should be configured with static mapping of both hubs
#. Spokes should be configured with both hubs as their NHS servers


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

Single Hub DMVPN with IKEv2
==================================

.. rubric:: Step 1: Configure the Phase 1 Proposal

::

  crypto ikev2 proposal <priority>
    encryption <encryption-algorithm>
    group <dh-group>
    integrity <integrity-algorithm>


.. rubric:: Step 2: Configure the Phase 2 Policy

::

  crypto ipsec transform-set <name> <enc> <hash>
    mode transport

.. rubric:: Step 3: Configure Authentication Details

::

  crypto ikev2 keyring <name>
    peer any
      address 0.0.0.0
      pre-shared-key <psk>

.. rubric:: Step 4: Configure IKEv2 Profile

::

  crypto ikev2 profile <name>
    match identity remote address <ip-or-wildcard>
    auth local pre-share
    auth remote pre-share


.. rubric:: Step 4: Configure IPSec Profile

::

  crypto ipsec profile <name>
    set transform-set <name>
    set ikev2-profile <name>

.. rubric:: Step 4: Configure Tunnel Interface

Configure Tunnel interface the same as in IKEv1 configuration


Single Hub DMVPN with IPv6
==========================

* Confgure 'ipv6 address' on LAN and Tunnel Interfaces
* All the IPv6 equivilent NHRP commands are the same as IPv4, just replace 'ip'
  with 'ipv6'
* If using IPv6 over the DMVPN but IPv4 on public interface, the IPv6 address
  should be specified as the private addresss and the public IPv4 address as
  the NBMA address
* For EIGRP, configure the tunnel interface as part of EIGRP process with
  'ipv6 eigrp <process-id>'


Dual Hub with Dual DMVPN
========================

* Configure multiple tunnel interfaces (one per DMVPN cloud)
  * Specify unique tunnel key (1)
  * Specify unique NHRP network ID (1)
