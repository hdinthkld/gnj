************************************
Cisco - VLAN Trunking Protocol (VTP)
************************************

.. _ccnp_switch_vtp:

VLAN Trunking Protocol (VTP) Overview
=====================================

- Cisco Proprietary
- Manages VLANs across the campus network
- Uses Layer 2 trunk frames
- Switches running as servers store the VLAN info in a "vlan.dat" file on the local switches flash memory.
- VLAN information maintained after poweroff
- By default switches will not have any domain name set, known as the "Null" domain
- Any switch that hears a summary advertisement sent non-securely will configure itself with the contained information.

.. _switch_vtp_versions:

VTP Versions
============

- Valid versions are 1,2 and 3
- Versions are not fully interoperable, recommended to use same on all switches
- Version is the default version
- Switches will attempt to change to version 2 from version 1 if they see a V2 advertisement.   Known as "VTPv2 Capable"
- Switches running VTPv3 will send scaled down advertisemenets if it hears a VTPv1 switch. Extended VLANs not included

VTPv2 Features
--------------

- Version dependant transparant  mode
- Consistency checks
- Token ring support
- Unrecognised TLV support

VTPv3 Features
--------------

- Extended VLAN range
- Enhanced Authentication
- Database Propagation
- Primary and Secondary Servers
- Per-Port VTP


VTP Domains
===========

- An Area with common VLAN requirements
- Switches only belong to one VLAN domain at a time
- Advertisement contents

  * Domain Name
  * Revision Number
  * Known VLANs
  * Specific VLAN Parameters

- Advertisements are used to notify other switches of VLAN changes

VTP Modes
=========

- Server

  * Full control over VLAN info
  * All VTP information advertised to other switches
  * Default mode for Cisco switches

- Client

  * Cannot make changes to VLANs
  * Lisens for VTP advertisements
  * Relays advertisements out of all trunk links

- Transparent

  * Does not participate in VTP
  * Version 1 will not relay advertisements
  * Version 2/3 will relay advertisements
  * Local VLANs not propagated

- Off

  * Switch does not participate in VTP
  * Will not relay advertisements

VTP Advertisements
==================

- Can be either Summary, Subset or Request advertisements
- VTP versions are not fully backward compatible
- By default version 1 advertisements are used
- Advertisements sent as multicast frames
- Advertisements relayed by all switches except those not running VTP ("Off" mode)
- Advertisements sent non-secure by default, a password can be configured
- Revision number used to track most recent information, starting at 0 (zero)
- Advertisemenets "usually" originate from a switch running in "server" mode
- "Client" switches will also originate advertisements on first boot

Advertisement Types
-------------------

- Summary

  * Servers send advertisemenet ever 300 seconds and on a VLAN change
  * Contains info about the VTP domain and revision number
  * For each VLAN change, a subset advertisemenet will follow

- Subset

  * Sent after each VLAN change, list specific changes made to the VLAN database

- Request
  
  * Sent from clients who are missing VLAN information

VTP Synchronisation
===================

- Switches will overwrite local VLAN ddata if a VTP advertisement contains a newer revision number
- Ensure new switchhes have a lower revision number by resetting to 0 by the following methods

  * Change VTP mode on the switch to transparent then back again
  * Change VTP domain to "non-existant" one and back to the original one

- An old switch replacing "live" information on a production network is referrred to as a "VTP Synchronisation Problem"
- When first booted even "Client" switches can replace live VLAN information as they send out summary advertisements. 
  This can cause even "server" switches to replace their information
- As "end-to-end" VLANs are now considered a legacy design model, Cisco recommends all switches be set to either "off" or "transparent"

.. _switch_vtp_pruning:

VTP Pruning
===========

- Assists with limiting size of broadcast domain when a VLAN is not used
- Avoids having to manually remove VLANs from a trunk
- Extension to VTP version 1 using additional VTP message type
- Switches will still run a spanning-tree instance even if the VLAN is pruned
- Disabled by default
- Enabling on the server also enables on all switches in the domain
- By default VLANs 2-1001 are eligible for pruning, list can be modified
- VLAN 1 and 1002-1005 can never be pruned
- No affect on transparent mode switches

VTP Configuration
=================

**Set VTP Version**

::

  vtp version {1 | 2 | 3}

**Set VTP Management Domain**

*Up to 32 characteers, no spaces*

::

  vtp domain <name>

**Set VTP mode**

::

  vtp mode { server | client | transparent | off }

**Set VTP Password**

*Password is never sent, only a hash is calculated*
*Hidden password is not shown inn the configuration, only a hash*

::

  vtp password <password> [ hidden | secret ]

**Verify VTP Status**

::

  show vtp Status


**Define this switch as the VTP Primary Server (Version 3 only)**

::

  vtp primary [force]

**Enable VTP Pruning**

::

  vtp pruning

**Define VLANS eligible for pruning**

::

  interface <name>
    switchport trunk pruning vlan {{[add | except | remove]} <vlan-list>} | none }

VTP Troubleshooting
===================

::

  show vtp status
  show vlan brief
  show interface <name> switchport
  show interface <name> pruning

Alternatives to VTP
===================

GARP VLAN Registration (GVRP)
-----------------------------

- Standards based VLAN management protocol for IEEE 802.1Q trunks
- Defined in IEEE 802.1D and 802.1Q (Class 11)
- Not supported by Cisco Catalyst Switches