****************************
Cisco - Multilayer Switching
****************************

Inter-Vlan Routing
==================

- VLANs are isolated broadcast domains
- Layer 3 device is required to pass traffic between them
- Traditional Inter-VLAN routing techniques

  * Inter-VLAN Router
    
    * Seperate interface per VLAN
    * Not scaleable beyond a few VLANs

  * Router with trunk interface
    
    * Single Interface used for all VLANs
    * "Router On A Stick" or " One-Armed Router"
    * Interface becomes the bottleneck

Routing And Switching functions using a multilayer switch (MLS)
===============================================================

- SDM templates may need changing depending on IPv4/IPv6 requirements
- Layer 3 interfaces can also be configured in an EtherChannel

Interface Types
---------------

- Layer 2 - Switching occurs between interfaces that are assigned as layer 2 on VLANs and trunk
- Layer 3 - Layer 3 switching occurs between any interface which support an assigned layer 3 address
- SVI - Switched Virtual Interface, A Layer 3 address assign to represent the entire VLAN

Steps To Configure Inter-VLAN routing
-------------------------------------

- Enable IP routing on the switch
- Configure Static and/or Dynamic routing
- Configure Layer 2 ports
- Configure Layer 3 ports
- Configure SVIs

Configuring an SVI interface
----------------------------

- Create VLAN
- Configure SVI (VLAN) Interface with assigned IP address

SVI Autostate
-------------

- By default SVI interface will not show as active until one or more ports in the correct VLAN are also active
- Can be overridden in special circumstances (E.g. port mirroring)

Multilayer switching with CEF
=============================

- Cisco Expres Forwarding (CEF) is the current generation of Cisco multilayer switches
- Used to forward packets based on Layer 3 and 4 information 
- Topology-Based MLS

Traditional MLS
---------------

- Dual effort between route processor (RP) and switching engine (SE) 
- "Route Once and Switch Many"
- "Shortcut Pathh" avoided subsequent packets needing to goto the RP
- Known as "NetFlow Switchin" or "Route Cache Switching"
- Used on Legacy Hardware

  * Catalyst 6000 supervisor 1/1a
  * Multilayer Switch Feature Card (MSFC)
  * Catalyst 5500 with Route Switch Module (RSM)
  * Catalyst 5500 with Route Switch Feature Card (RSFC)

CEF Overview
------------

- High Performance Packet Forwarding
- Runs By default on modern Cisco switches
- Takes advantage of special Hardware
- Functional Blocks

  * Layer 3 Engine (Routing Table, ARP Table)
  * Layer 3 Forwarding Engine (FIB, Adjacency Table)

- Layer 3 Engine build routing information that Layer 3 Forwarding engine can use to switch packets

- Some packets cannot be CEF switched and are sent to L3 Engine, known as "CEF punt"

  * Entry not located in the FIB
  * FIB table is full
  * IP TTL expired
  * MTU Exceeded and Fragmentation needed
  * ICMP Redirect
  * Unsupported Encapsulation Type
  * Packets need to be tunnelled, compressed or encrypted
  * Access List logging
  * NAT Operation required

Forwarding Information Base (FIB)
---------------------------------

- A reordered copy of the routing table so that most specific route is listed first
- Contains next-hop for each entry
- Host routes (/32) are included for directly connected (adjacent) hosts
- Layer 3 engine sends updata to FIB when topology changes
- FIB also updated when next-hop or MAC address changes or is aged out
- Version number and nuber of changes (Epoch) are tracked

CEF Optimization
----------------

- Accelerated CEF (aCEF)
  
  * Multiple Layer 3 Forwarding Engine
  * Individual line cards on chassis switches
  * Only used a portion of the FIB known as a "FIB Cache"
  * CEF accelerated on the line card but not at sustained wire speed

- Distributed CEF (dCEF)
  
  * FIB Completely distributed among multiple Layer 3 Forwarding engines
  * Catalyst 6500 line cards support dCEF, each has own FIB an Forwarding Engine
  * Central Layer 3 Engine maintains routing table and downloads FIB to each line card


Adjacency Table
---------------

- Routers usually maintain Routing Table (Layer 3) and ARP table (Layer 2) seperately
- FIB maintains the Layer 3 next hhop for each entry
- FIB also has Layer 2 information for next hop, the area of the FIB is known as the "Adjacency Table"
- Adjacencies kept for each next hop router and each directly connected host
- Entries contain both IP address and MAC address
- Built from the ARP table
- Updates as new ARP replies are received
- Unknown entries (CEF Glean) must be sent to Layer 3 Engine with packets being dropped until the entry is known to avoid exhausting the input queue

- Adjacency Types can be one of Null, Drop, Discard or Punt

- Null Adjency

  * Packets to be sent to null interface
  * Absorbs packet without forwarding

- Drop Adjacency

  * Switches packets that cannot be forwarded
  * Silently dropped
  * Used for encapsulation failures, unsupported protocols, no valid route, no adjancy, etc

- Discard Adjacency

  * Packets to be dropped due to access list or other policy action

- Punt Adjacency

  * When packets must be sent to the Layer 3 Engine
  * Incomplete adjacencies (NO_ADJ)
  * Incomplete ARP resolution (NO_ENCAP)
  * Unsupported Packet Features (UNSUPP'TED)
  * ICMP Redirects (REDIRECT)
  * Packets destined for switch interfaces (RECEIVED)
  * IP Options (OPTIONS)
  * Access list evaluation failure (ACCCESS)
  * Fragmentation Failures (FRAG)

Packet Rewrite
--------------

- Last step prior to forwarding requires packets to be rewritten
- Layer 2 destination address changes to next hop MAC address
- Layer 2 source address changes to outbound L3 swiitch interface mac
- Layer 3 IP TTL decremented by 1
- Layer 3 IP Checksum updated
- Layer 2 Frame checksum updated
- CEF uses dedicated hardware for efficient rewrite and table lookups

Multilayer Switch Configuration
===============================

Global Configuration
--------------------

**Create a VLAN**

*NOTE: Should be done before creating an SVI/VLAN interface*

::

  vlan <id>
    name <string>

**Disable/Enable CEF**

*NOTE: CEF cannot be disabled on all platforms*

::

  [no] ip route-cache cef
  [no] ip cef

Interface Configuration
-----------------------

**Configure interface to operate at Layer 2**

::

  interface <name>
    switchport

**Configure interface to operate at Layer 3**

::

  interface <name>
    no switchport


**Create an interface to route in/out of the VLAN (SVI)**

::

  interface Vlan<id>
    ip address <ip> <mask> [secondary]
    no shutdown

**Exclude interface from affecting SVI autostate**

::

  interface <name>
    switchport autostate exclude

Verification Commands
---------------------

**Verify Switch Port Mode**

::

  show interface <name> switchport

**Display FIB Table for Specific Interface**

::

  show ip cef [ <name> | vlan <id> ] [detail]

**Display FIB Table by IP Prefix**

::

  show ip cef [<prefix> <mask>] [longer-prefixes] [detail]

**Display Adjacency Table**

::

  show adjacency [<interface-name> | vlan <id>] [summary | detail]

**Display CEF entries without valid ARP (CEF Glean)

::

  show ip cef adjacency glean

**Display Statistics for CEF Drop reasons**

::

  show cef drop

**Display Statistics for packets not processed by CEF**

::

  show cef not-cef-switched

**List configured VLANs**

::

  show vlan

**Display IP Information about switch interfaces**

::

  show ip interface <name>

**Display Summary of Layer 3 Interfaces**

::

  show ip interface brief


**Display Entire FIB**

::

  show ip cef

