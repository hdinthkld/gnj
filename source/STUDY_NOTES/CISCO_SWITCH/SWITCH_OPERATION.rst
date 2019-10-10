****************************************
Cisco - Switch Operation
****************************************

.. _ccnp_switch_switch_operation:

Layer 2 Operation On Legacy Networks
===========================================

- A Network where all hosts share the same bandwiddth is called a shared media network.
  Used in legacy netwroks with hubs and CSMA/CD Scheme
- Carrier Sense Multiple Access with Collection Detect (CSMA/CD) determine when devices 
  were allowed to transmit
- Collision occurs when two or more hosts transmit at the same time.  Hosts must operate
  in half-duplex mode (Transmit or Receive, no both)
- Hosts  must "back off" for a random perioud befoe trying to transmit again
- All frames must be processed by all hosts, even those not addressed to them and also frames with errors

Layer 2 Operation On Switched Networks
======================================

- Switches operate at layer 2, unlike hubs that operate at layer 1
- Switches decide where to send frames based on their destination MAC addressed
- Media no longer shared with other hosts as long as the port is running in full-duplex
- Host isolation

  * Each switch port is its own collision domain
  * Host connections operate in full duplex mode
  * Bandwidth on a port is no longer shared
  * Switches check frames for errors, dropping or forwarding as appopriate (Store And Forward)
  * Broadcast Traffic can be limited to a given threshol (Storm Control Feature on Cisco Switches)
  * Intelligent Filtering/Forwarding Features available

Transparent Bridging
====================

- Layer 2 switch is a multi-port transparent bridge, each port an isolated LAN segment
- Frame forwarding based entirely on the destination MAC address in the frame
- A switch must be told on what port an address exists or learn it for itself

- Dynamic Learning/Forwarding

  * Switch Listens To Incoming Frames
  * Source MAC address along with incoming port and VLAN are stored in a table kept in the switches memory
  * Forwarding table is checked for destination MAC address, port and VLAN.  If found frame is forwarded out of that port only
  * When not found, frame is referrred to as an "Unknown Unicast" and is "Flooded" out of all ports in same VLAN excep the originating port
  * After a response is received the switch will know the port and VLAN so no further flooding is needed for that MAC address
  * Broadcast and Multicast Frames cannot be learned so are always flooded

Layer 2 Catalyst Switch Decision Process
========================================

- Coming frame placed in switch ports ingress queue

  * Queues can have different priority or service levels to allow time sensitive data to be processed first

- For each frame, as its pulled from the ingres queue 3 decisions are made

  * Layer 2 Forwarding Table (or "CAM Table") is checked for port and vlan using the destination MAC as the index
  * Security ACLs stored in the "TCAM" are checked to see if a frame should be forwarded
  * QoS ACLs are checked to classify, policy or control rate of traffic and mark QoS Parameters in the frame
  * Single parallel table lookup is used for all decisions

- Once all table lookups performed, frame is placed in an appropriate egress queue, determined by QoS values
  in the frame or passed along with the frame.

Multi-Layer Switch Operation
============================

- Types of Multi-Layer Switching (MLS)

  * Route Caching (1st Generation MLS)
  * Topology Based (2nd Generation MLS)

- MLS is a method of forwarding frames based on Layer 3 & 4 information
- Layer 2 switching is still performed
- Cisco IOS Catalyst Switches only support 2nd Generation Topology Based MLS

- Route Caching (1st Generation MLS)

  * Required a Router Processor (RP) and Switch Engine (SE)
  * RP processes first packet in flow to determine destination
  * SE listens and caches result for use on subsequent packets
  * Known By NetFlow LAN, Flow-Based or Demand-Based switching
  * "Route Once, Switch Many"
  * Still Used to generate traffic flow info and statistics

- Topology Based (2nd Generation MLS)

  * Specialised Hard with Distinct RP and SE functions
  * RP takes Layer 3 routing info to a single database of the network topology, stored in hardware
  * Database is consulted and packets forwarded at high rates by the SE
  * Hardware database can be updated with no performance penalty
  * Known as "Cisco Express Forwarding" (CEF)
  * Routing process downloads routing table into a area of hardware known as the "Forwarding Information Base" (FIB)
  * The control plane includes the RP
  * The data plane exists in the SE

Layer 3 Catalyst Switch Decision Process
=======================================

- Packet is pulled off the ingress queue, inspecte for Layer 2 and Layer 3 destination addresses
- Deciding where to forward a packet

  * Layer 2 Forwarding Table (CAM) is checked to see if destination is a port on the actual switch. Determines is frame should be layer 3 switched
  * Layer 3 Forwarding Table (FIB) is checked using destination IP as a index. Longest match is found (Address and Mask), Next-Hop Layer 3 Address along
    with Layer 2 MAC address, Egress port an Vlan ID is obtained to avoid further lookups

- Deciding How to forward a packet
  * Security ACLs for inbound and outbound are compiled into TCAM to allow a decision on a single lookup
  * QoS ACLs for classification, policing and marking all performed as a single lookup on QoS TCAM

- Packet is placed in appropriate egress queue on the correct port
- Packet will be rewritten just like on a router (eg. checking DST MAC, decrementing TTL)
- Entire ethernet frame is rewritten in hardware

Multi-Layer Switching Exemptions
================================
Any of the list below are processed more slowing in software

- ARP Request/Replies
- IP Packets requiring a router response (TTL expired, Max MTU, Fragmentation, etc)
- IP Broadcasts received as Unicast (DHCP Requests/IP Helpers)
- Routing Protocol Updates
- CDP Packets
- Packets needing encryption
- Packets requiring NAT
- Legacy Multiprotocol Packets (IPX, Appletalk, etc)

Content Addressessable Memory (CAM)
===================================

- Used for layer switching
- Source MACs are recorded as frames arrive on a switch port
- If a MAC moves port, new entry recorded then old entry deleted
- CAM table space is finite, stale entries removed afer a specified aging time time (default : 300 seconds/5 minutes)

Managing the CAM Table
======================

**Show the current contents of the CAM table**

::

  show mac address-table [<options>] - Recent IOS, not 4500/6500
  show mac-address-table [<options>] - Pror to 12.1(11)EA1


**Check the size of the CAM Table**

::

  show mac address-table count

**Adjust CAM table time for removing stale elements**

::

  mac address-table aging-time <seconds>

**Add static CAM table entries**

::

  mac address-table tatic <MAC> vlan <ID> interface <interface-name>

**Clearing CAM Table Entries**

::

  clear mac address-table dynamic  [<options>]

Ternary Content Addressable Memory (TCAM)
=========================================

- Packets are evaluated using a single lookup on a table implemented in hardware
- Avoids latency of traditional routing when matching, filtering or controlling specific traffic
- Most switches have multiple TCAMs so inbound/outbound and QoS ACLs are evulated simultaneously or in parallel with layer 2/3 
  forwarding decisions

- Components of TCAM operation in IOS

  * Feature Manager (FM) compiles/merges ACLs into the TCAM table
  * Switching Databae Manager (SDM) configures or tunes the TCAM partitions to optimise for specific functions

- TCAM is fixed on the 4500/6500 platforms, cannot be repartitioned through SDM

- TCAM Structure

  * Extends the CAM table concept
  * Uses three values (0,1, "Don't care") known as a terniary combinatio
  * Entries may up a value, mask and result (VMR) combination
  * Values and Masks are 134-bit quanities
  * Results are the action that should be taken after lookup (e.g. permit/deny, Qos Policier, Next-Hop, etc)
  * If IPv6 is used some address compression is required to store in the TCAM
  * TCAM is organised by mask with associated value patterns
  * Operations involving any other than exact matches requires use of an "Logical Operation Unit" (LOU) which are limited in number
  * Exceeding the numbber of LOU's require ACE's to be expanded so they only may use of the "eq" operator

- The TCAM cannot be manipulated directly, to see the current utilisation use

::
  
  show platform tcam utilisation

Managing Switching Table Sizes
==============================

- The Catalyst 4500/6500 has ample resources for core, distribution or access layer switches and does not allow modification
  of table sizes
- Other platforms should have their resource assigned as follows
  
  * Switches running at Layer 2 should have a larger CAM table
  * Switches running at Layer 3 should have a larger FIB table

- The switching database manager (SDM) managed how the switches memory is partitioned and these are defined as templates

  * Desktop (Default, Access, VLAN, Routing)
  * Dual-Ipv4-And-IPv6
  * Indirect-Ipv4-And-IPv6

- Display the current table sizes

::

  show sdm prefer

- Change the current template (requires reboot)

::

  sdm prefer <template-name>

Media Access Control (MAC) Addresses
====================================
- "Unique" address assigned to a network interface
- Sometimes called hardware address or burned-in address (BIA)

- Can be assigned by the manufacturer (Globally Unique)

  * First 3 octets (24 bits) are the organisationally unique identifier (OUI)
  * Last 3 octets (24 bits) are NIC specific
  * Bit 1 of 1st octet set to 0 (zero) for globally unique

- Can be assigned by an organisations admin (Locally Administered)

 * Bit 1 of 1st octet set to 1 (onee) for locally administered

- Total size of MAC Address is 4 bits
- Globally unique MAC addresses maanged by the IEEE
- MAC Addresses are written in trannsmission order, least significant bit first (Left-To-Right) as done for ethernet
- Token ring uses most significant bits first