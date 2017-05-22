###################################
Dynamic Multipoint VPN Fundamentals
###################################

Introduction
============

Dynamic Multipoint VPN (DMVPN for short) is designed to simplify and make it
easier to maintain full-mesh connectivity as the network grows.

Allows for both hub-to-spoke and spoke-to-spoke (Phase 2) connectivity



Functional Components
=====================

* mGRE
* NHRP
* IPSec Proxy Instantiation
* Hub and Spoke devices

How it works
============

NHRP
^^^^

* Layer 2 Protocol
* "ARP" for DMVPN
* Resolves the NBMA (Public Address) given the peers DMVPN "Cloud" IP
* Spokes will register their NBMA address with the hub (Next Hop Server)
* After NHRP has registered information, the  persistant tunnel is established
  between hub and spoke
* When a spoke needs to access a network at another spoke site it will query the
  hub via NHRP
  for the spokes NBMA address, providing the next hop IP as listed in the
  routing table.
* The clients must be preconfigured with both the NBMA and Public IP of the Hub
* No concept of primary/secondary hubs, all are equal and routing determines
  which hub the spokes connect to.

.. todo:: what is the purpose of the network id?

mGRE
^^^^

* Multipoint GRE
* Layer 3 Protocol
* Adds 28 byte header (as compared to 24 bytes for GRE)

.. todo:: What is the purpose of the tunnel key?

Working Process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. rubric:: Initial Tunnel Establishment

* Hub is configured and makes itself available for spoke sites to connect
* Spoke sites boot up and establish a connection with their preconfigured hub
* Once hub tunnel is established the spoke registers with the hub via NHRP it's
  NBMA address
* The hub stores the NBMA address along with the spokes private IP in a lookup
  table
* Routing information is shared between hub and spoke as normal

.. rubric:: Spoke needs to send traffic to another spoke's network

* Spoke consults it's routing table to determine the exit interface for traffic
* If it finds it's via the DMVPN tunnel it consults it's NHRP database to see
  if it knows where to send traffic
* If it already has an entry it will use the existing tunnel to communicate
  directly with the the spoke
* If no information is found, the spoke will send an NHRP query to the hub
  asking for the other spokes NMBA address
* At the same time, the spoke will forward the traffic to the hub until a NHRP
  response is received
* Once the NHRP response is received, the spoke will establish an IPSec tunnel
  with the other spokes NBMA address
* After the tunnel has been established the original spoke will now send packets
  directly to the destination spoke
* When the receiving spoke needs to send packets to the original spoke it will
  make it's own NHRP query to the spoke
* As the tunnel has already been established the receiving spoke will now know
  to use the existing IPSec connection for reply packets


DMVPN Phases
^^^^^^^^^^^^
* 1, 2, 3

.. rubric:: Phase 1 - Spoke-To-Hub Connectivity Only

* Spoke registers with Hub on intial connection
* Inter-Spoke connectivity always goes via the Hub
* No dynamic tunnel is created

.. rubric:: Phase 2 - Dynamic Spoke-To-Spoke Connectivity

* Spoke registers with hub on initial connection
* When a spoke needs to comunicate with another hub, a dynamic tunnel is
  created based on NHRP information
* Only the hub is queried for NBMA information

.. rubric:: Phase 3 - Improvements to NHRP query response

* 'ip nhrp redirect' on hub
* 'ip nhrp shortcut' on spoke
* Spokes registers with the hub
* When spoke needs to send packets to other spoke, initial packet and NHRP query
  sent to hub
* Hub will redirect NHRP query to the other spoke (ip nhrp redirect)
* Receiving spoke will update it's NHRP database with the received information
  and reply directly to the original spoke.

  .. todo:: What about 'ip nhrp shortcut'?

Limitations
============

* Distance Vector Algorithms (e.g. RIP or EIGRP) should have split horizon
  disabled on the tunnel interface so routing updates can be redistributed via
  the same multiple point interface.
* When OSPF is used the tunnel interface must be configured as a 'broadcast',
  not 'point-to-multipoint' network so that next hop information is not the hub
* Until dynamic tunnel is established, all traffic will go via the hub
* Supported on IOS devices only (not supported on ASA/PIX)
* Works in IPSec transport mode
* Failure of a single hub will cause the entire DMVPN to eventually fail

Advantages
==========

* No need to manually configure inter-spoke communication, new spokes can be
  added as required with only configuration to inform new spoke of the hub(s)
* Unicast and Multicast routing support
* Dynamic routing
* Dynamic IP addressing on spoke
* Behind NAT devices supported
* Can be used without IPSec
* Support VRF
* Supports QoS

Hub Requirements
================

* Must have a fixed IP address
* No need for spoke sites to be configured in advance
* All IPSec configuration (except traffic specifies and peers) must be configured in advance

Spoke Requirements
==================

* Can have either a static or dynamic IP address
* Must be preconfigured with the primary hub (as well as any secondary hubs)
* All IPSec configuration (except traffic specifies and peers) must be configured in advance

Intermediate Network requirements
=================================

Fault Tolerance and High Availability
=====================================
* Dual Spoke

Terminology
===========

* NHRP
* NHS - Next Hop Server (the hub router in a DMVPN)
* NBMA - The public IP of a device operating in a DMVPN cloud
