********************************
Cisco - Aggregating Switch Links
********************************

.. _switch_aggregation_overview:

Switch Port Aggregation with Ether Channels
===========================================

- Under normal circumstances STP will only allow a single link to forward traffic
- Etherchannels bundle multiple parallel links to at a single link
- Between 2 - 8 Fast, Gigabit or 10Gb ethernet interfaces can be bundled to give full-duplex bandwidths of:
  
  * FEC - Up to 1600Mbps
  * GEC - Up to 16Gbps
  * 10GEC - Upto 160Gbps

- The bundled link acts as a single access or trunk link
- When created on Cisco devices the interface type is known as a "Port-Channel"
- Traffic is distributed based on the defined load balancing algorithm
- Traffic will be distributed across the links but not necessarily equally
- Muliple links increase redudancy with the STP port cost adjusted according
- STP reconvergence is only needed as a last resort as long as it least one link in the EtherChannel is available
- If the platform supports switch stacking, the EtherChannel can be distributed across multiple stack members
- Higher end switches that support Virtual Switching System (VSS) can form the EtherChannel across 
  the switch chassis's into a Multi-Chassis EtherChannel (MEC)
- All interfaces that are combined into a single EtherChannel must have identical settings
- Etherchannel can be used on both Switched (Layer 2) and  Routed (Layer 3) ports

.. _switch_aggregation_lb:

Distributing Traffic In An EtherChannel
=======================================

- Traffic is distributed over all the links in the bundle in a deterministic way
- Traffic is not ballanced equally depending on the trafffic flow
- Hashing Algorithm used takes into account

  * Source/Destination IP addresses
  * Source/Destination MAC Addresses
  * Source/Destination TCP/UDP Port Numbers

- Output of the algorithm determines the link to be used
- An XOR operation on one or more lower order bits is used depending on the number of links in the bundle

  * 2 Links - 1-bit XOR operation
  * 4 Links - 2-bit XOR operation
  * 8 Links - 3-bit XOR operation

- Host-To-Host communication always goes over sam3 link as long as addresses remain constant
- Load balancing is globally defined, not per port
- All models can load balance using MAC and IP addresses
- Higher End switches (4500, 6500) can also use TCP/UDP port numbers
- When using a router-to-router (layer 3) connection, consider using IP instead of MAC addresses
  for better distribution
- If non-IP traffic is seen, load balancing based on MAC addresses is used automatically

EtherChannel Negotiation Protocols
==================================

- EtherChannels can be negotiated for use when needed
- Two protocols are available
  
  * Port Aggregation Protocol (PAgP) - Cisco Proprietary
  * Link Aggregation Control Protocol (LCAP) - Standards Based

- When negotiation is used, an additinal 15 second delay may occur
- Configure the negotion mode to "on" to not use negotiation


.. _switch_etherchannel_pagp:

Port Aggregation Protocol (PAgP)
--------------------------------

- Cisco Proprietary protocol
- EtherChannel can only be formed when identical parameters configued
- Member port config is modified if EtherChannel is changed
- Port modes

  * Desirable - Actively try to establsh an EtherChannel
  * Auto - Form an EtherChannel but only if requested to do so

.. _switch_etherchannel_lacp:

Link Aggregation Control Protocol (LACP)
----------------------------------------

- Standards Based on IEEE 802.3ad
- Up to 16 links can be configured in the EtherChannel but only 8 active at any one time
- Switch which has the lowest system priority (2-bytes) chooses which ports are active in the EtherChannel
- Links selected are based on port priority
- Unused ports put in standby state
- Port modes

  * Active - Try to establish EtherChannel with neighbour switch
  * Passive - Wait for neighbour switch to request negotiation of EtherChannel


.. _switch_aggregation_guard:

Avoiding Misconfiguration with EtherChannel Guard
=================================================

- Enabled by default
- Issues unlikely to occur when PAgp/LACP are used
- Likely to be an if configured to EtherChannel unconditionally
- Ports are port inn errdisable state if misconiguration (i.e. cabling) issue detected

Troubleshooting An EtherChannel
===============================

- Ensure both switches are configured for unconditional etherchannel or one should actively to negotiate
- Ensure configuration is consistent across all ports to be bundled on both switches
- Check cabling is correct


EtherChannel Configuration
==========================

Global Configuration
--------------------

**Set the Load Balancing Method**

::

  port-channel load-balance <method>

**Verify Load Balancing Method**

::

  show etherchannel load-balance

**Disable/Enable EtherChannel Guard**

::

  [no] spanning-tree etherchannel guard misconfig


**Check if ports are disabled due to EtherChannel Guard**

::

  show interface status err-disabled

PAgP EtherChannel Configuration
-------------------------------

::

  interface <name>
    channel-protcol pagp
    channel-group <id> mode {on | {{auto | desirable} [non-silent]}}

LACP EtherChannel Configuration
-------------------------------

::

  lacp system-priority <priority>

  interface <name>
    channel-protocol lacp
    channel-group <id> mode {on | passive | active}
    lacp port-priority <priority>

Troubleshooting EtherChannels
-----------------------------

**Enable interface following misconfiguration**

::

  interface port-channel<id>
    shutdown
    no shutdown

**Status/Validation Commands**

::

  show etherchannel summary
  show etherchannel {port | port-channel | detail}
  show running-config interface <name>
  show inteface <name> etherchannel
  show {pagp|lacp} neigbor
  show lacp sys-id
