.. _ccnp_switch_campus_design:

****************************************
Cisco - Enterprise Campus Network Design
****************************************

Collision Domain
================

- A shared network segment upon which multiple hosts compete for usage
- Collisions occur when two or more hosts attempt to transmit at the same time
- Network segmentation is used to reduce the sie of a collision Domain
- Switches allow for micro segmentation where each port on the switch is its own collection when running at full duplex

Broadcast Domain
================

- Broadcast traffic is seen by all hosts on the segment
- Excessive broadcast traffic causes performance problems and can monopolise bandwidth
- Size of broadcast domains can be reduced by segmenting networks into VLANs

VLAN
====

- Defines the boundary of a broadcast domain
- Passing traffic between VLANs requires a Layer 3 device

Predictable Network Model
=========================

- Low maintenance
- High Availability
- Designed around traffic flows, not type of traffic
- Arranged so users are a consistent distance from needed resources

Hierarchical Network Design
===========================

- Efficient, Intelligent, Scaleable, Easily Managed
- Distinct Layers of devices

  * Core
  * Distribution
  * Access

- Traffic Flow type

  * Local
  * Remote
  * Enterprise

Access Layer
------------

- End user connectivity at Layer 2 (VLANs)
- Building Access Switches
- High Port Density with Low Cost Per Port
- Scaleable Uplinks with High Availability
- Converged Network Services (Data, Voice, Video)
- Security Features and QoS

Distribution Layer
------------------

- Interconnects the Access and Core Layers
- Building Distribution Switches
- Aggregation of Multiple Access Layer Switches
- High Layer 3 Routing Throughput, Layer 3 Boundary For Access Layer VLANs
- Security And Policy Based Functions
- QoS
- Scaleable and redundant high speed links to Core and Access layers

Core Layer
----------

- Connectivity Between Switches in the Distribution Layer
- Backbone
- Switches traffic as efficiently as possible
- Very High Layer 3 Routing Throughput
- No unnecessary Packet Manipulation
- Redudancy and Resilience
- Advanced QoS

- Smaller networks may combine core and distribution layers, known as a collapsed core. Not an independent building block

Best Practices
--------------

- Design each layer with a Pair of switches
- Connect each switch to next layer with redudant links
- Connect each pair of distribution switches with at least one link
- Do not connect access layers unless a stack or chassis switch
- Do not extend VLANs beyond distribution switches
  
Modular Network Design
======================

- Permits the network to grow in a logical predictable manner
- A switch block is a group of access layer switches with one or more distribution layer switches
- Takes redudancy and scaleability into account
- Core/Backbone layer used to connect all switch blocks
- Other elements (such as a Datacenter block) can be added as needed

Switch Block Sizing
-------------------

- Traffic Types andd Behaviour
- Size and number of common workgroups

Oversized switch block problems
-------------------------------

- Bottlenecks at distribution layer (CPU, Memory, Throughput, etc)
- Broadast/Multicast traffic slows the switches in the switch block
    
Switch Block Redundancy
-----------------------

- Multiple Power Supplies on different power sources
- Multiple distribuion switches
- Use a FHRP on the destribution switches (HSRP, VRRP, GLBP)
- Avoid Layer 2 loops where possible
- Do not use same VLAN across multiple access switches

Network Core
============

- Required to connect two or more switch blocks
- As efficient and reliable as possible
- Links between core and distribution shoul be layer 3
- Consider average aggregate link utilisation when scaling links
- Deploy two multilayer switches (MLS) as a dual-core for Redundancy
- Add more core switches to form a multi-node core when needed, fully mesh if possibble
- Number of core switch peerings is limited only by number of distribution layer switches 

Choosing Cisco products to use at each layer
============================================

- Access Layer - High Port Densityy, Low Cost

  * 2960-X, 3650, 3850 with 48 ports each
  * 4500E For a single chassis switch

- Distribution/Core Layer - High Layer 3 Switching Throughput, High Density, High Bandwidth Ports

  * 4500-X, 45000E, 6807-XL
  * 3750-X For smaller switch blocks/networks


Traffic Flow Paths
==================

- Local (Access Layer Only)
- Remote (Access To Distribution)
- Enterprise (Access To Distribution To Core)
