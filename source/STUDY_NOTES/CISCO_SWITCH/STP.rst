************************************************
Cisco - Traditional Spanning Tree Protocol (STP)
************************************************

.. _ccnp_switch_stp:

IEEE 802.1D Overview
====================

- STP is defined IEEE 802.1D
- Provides link redundancy and Automatic Failover Recovery

Transparent Bridge Operation
----------------------------

- Listen for frames coming into its ports
- Builds table of MAC addresses based on the information stored in the frames (Source MAC address)
- Updates table on any MAC address moves
- Broadcasts flooded out all except source port
- Frames are not modified by the bridge

Bridging Loops
--------------

- Occur when a single frame continuously goes around and round between two or more switches.
- There is no Layer 2 equivilent of TTL unlike at Layer 3 so the frame cannot be stopped
- Bridging loops without spanning-tree can only be stopped by unplugging cables

Preventing Loops with Spanning-Tree-Protocol
--------------------------------------------

- Bridging loops occur due to parrallel switches not being aware that other switches exist
- STP developed so that switches are aware of each other and to enable use of redudent switches
  or switch paths
- STP is used to negotiate a loop free path
- Any loops are discovered before being made available for use
- Each switch executes the STP algorithms independent of other switches
- Redudant paths are put into a "Blocking" or standby state to prevent them forwarding frames


Bridge Protocol Data Units (BPDUS)
----------------------------------

- A form of data message used for inter switch communication
- Sent out of all switch ports configured for trunking
- A unique MAC address is used on each port if supported by the switch platform
- Destination is defined as the multicast address 0180.c200.0000
- A BPDU can be either the type "Configuration" or "Topology Change Notification" (TCN)
- The "Configuration" BPDU is used to notify other switches of a change to STP Configuration
- A TCN BPDU is used to announce a change on the network, such as a port coming up or down
- The exchange of BPDUS is used to elect a reference point for the network and build a stable topology
- BPDUs are sent out of all trunk ports every 2 seconds by default


Spanning Tree Election Process
==============================

Root Bridge Election
--------------------

- Root bridge is a common reference point for all switches
- Bridge ID is assigned to each switch composed of "Bridge Priority" (2 Bytes) and
  MAC address (6 bytes)
- On first power up switch sends BPDUs
- Once Root bridge is elected only it will sent configuration BPDUs
- The best root bridge is decided by lowest priority an lowest MAC address if a tie
- Default priority for Cisco switches is 32768.  can be set in incremeent of 4096

Root Port Election
------------------

- Decides the "best" path to reach the root bridge
- A path cost is calulated based on cost of all links leading to root bridge
- Only root path cost is carried in the BPDU, path cost is known only to local switch
- Switches modify root path cost as BPDU is propagated across the network
- Initial root path cost in root bridge BPDU is 0 (zero)
- Next switch adds path cost based on interface speed as BPDU is received
- Switch updaed BPDU with root path cost before relaying it on

**Cisco Path Cost Values**

=============== ========= ============
Interface Speed New Value Legacy Value
=============== ========= ============
4Mbps              250       250
10Mbps             100       100
16Mbps              62       63
45Mbps              39       22
100Mbps             19       10
155Mbps             14        6
622Mbps              6        2
1Gbps                4        1
10Gbps               2        0
=============== ========= ============

Choosing Best Designated ports
------------------------------

- A designated port is the path is used by the local segment to reach to the root bridge
- Only a single designated port exists per segment in order to avoid bridging loops
- The port with the lowest cumulative root path cost is choosen

Resolving a tie situation
-------------------------

- Lowest Root bridge ID
- Lowest Root path cost to root bridge
- Lowest sender bridge ID
- Lowest sender port id

Spanning Tree Port States
=========================

- Disabled

  * Not part of the normal STP Port progression process
  * Used for ports that are "Administatively Shutdown"

- Blocking

  * Port initialised
  * Receive only BPDUs

- Listening

  * Receive BPDUs
  * Send BPDUs

- Learning

  * Changes to this state after the Forward Delay timer has expired
  * Send/Receive BPDUs
  * Learn MAC addresses

- Forwarding

  * Full operation
  * Send/Receive BPDUs
  * Learn MAC addresses
  * Send/Receive data frames

Manually calculating STP topology
=================================

- Identify Path Cost on links
- Identify Root Bridge
- Select Root Ports (1 per switch)
- Select Designated Ports (1 per segment)
- Identify Blocking Ports

STP Timers
==========

- Hello Timer 

  * Defaults to 2 seconds
  * Frequency at which configuration BPDUs are sent by the root bridge
  * Locally configured Hello time is used to time TCN BPDUs

- Forward Delay Timer

  * Defaults to 15 seconds
  * Delay between a port moving from listening to learning state

- Max Age Timer

  * Defaults to 20 seconds
  * Time a BPDU is stored before being discarded
  * If  no more BPDUs received, switch assumes topology change mus have occurred

Topology Changes
================

- Annouced through TCN BPDUs
- Changes occur when

  * Ports move into the forwarding state
  * Port moves from forwarding/learning to Blocking state

- TCN BPDUs not sent if change was detected on a "PortFast" configured port
- TCN BPDUS sent ever hello time interval until acknowledgment received
- Upon received TCN BPDU, root bridg will

  * Send acknowledgement
  * Sets topology change flag in the config BPDU
  * Relay BPDU to all other switches

- Switches receiving BPDU with change flag set will redice their bridge table aging (default: 300 seconds) to
  the forward delay value (15 seconds) in order to cause MAC addresses to be flushed more quickly from the briding table.

- Topology changes can be either direct or indirect

Direct Topology Changes
-----------------------

- Changes detected on a switch interface
- Root bridge sends config BPDU with change flag set
- Other switches receive BPDU and shorten aging time
- Switches update root port accordingly
- Connectivity will will be 2 x forward delay timer (30 seconds)

Indirect Topology Changes
-------------------------

- Failure which does no cause link status to change
- No TCN BPDU sent
- Stored BPDU is flushed after Max Age time expires (Default: 20 seconds)
- Switch waits to receive BPDU from root bridge
- Port progresses through blocking, listening, learning to forwarding
- Upto 1 minute of connectivity loss could occur (2+15+15+20)


Insignificant Topology Changes
------------------------------

- Ports connected to end user devices by default will still cause TCN BPDUs to be generated
- Cause unnecessary flushing of CAM tables
- Results in more unknown unicast traffic
- "PortFast" is configured on ports connected to end user devices causing TCN BPDUs not to be sent
- "PortFast" causes ports to brought up directly into the "Forwarding" state

Types of STP
============

- STP originally designed to support only a single VLAN
- IEEE and Cisco approached STP differently

Common Spanning Tree (CST)
--------------------------

- IEEE 802.1Q standard
- Single instance for all VLANs
- CST BPDUS send over trunks using native VLAN untagged frames
- Does not allow use of redudant links

Per-VLAN Spanning Tree (PVST)
-----------------------------

- Cisco Proprietary
- Seperate STP instance for each VLAN
- Load balancing possible using redudant links, over different VLANs
- Requires use of ISL trunk encapsulation
- Interoperability issues with CST

Per-VLAN Spanning Tree (PVST+)
----------------------------------

- Cisco Proprietary
- Improves interoperability with PVST and CST
- Operates over both 802.1Q and ISL

Redudant Link Convergence
=========================

Cisco provides additional means in order to reduce the time taken to establish a stable topology following a failure:

- PortFast (Access Ports)
- UplinkFast (Access Layer Uplinks)
- BackboneFast (Redudant Backbond links

PortFast
--------

- Reduces listening and learning timers
- Port moves immediately to forwarding state
- Loop detection still in operation
- Disabled by default
- Can be enable globally for non-trunking links or manually per port
- TCN BPDUs not sent when link goes down
- Port looses PortFast status if BPDU is seen on the port

UplinkFast
----------

- Disabled by default
- Can be enabled globally, not per port
- Not supported on the root bridge
- Maintains one or more redudant root ports
- Upon failure of a link anoher upliink is brought into operation immediately
- If enabled, bridge priority changed to 491522 and port cost increased to 3000
- When failure detected, dummy multicast frames are sent for all stations in the CAM table so that other
  switches learn the new path as quickly as possible.

BackboneFast
------------

- Uses interactive process to detect alternative paths
- Root Link Query (RLQ) protocol is used to interact with directly connected switches
- When reply is received on non-root port, Max age timer immediately expires
- Reduces convergence delay from 50 to 30 seconds
