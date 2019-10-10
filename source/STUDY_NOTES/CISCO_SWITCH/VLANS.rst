************************
Cisco - VLANs And Trunks
************************

.. _ccnp_switch_vlans:

Virtual LANs (VLANs)
====================

Flat Network
------------  

- Layer 2 only switched Network
- Single Broadcast Domain
- Cannot contain redudant paths

What is a VLAN
--------------

- A VLAN is a single broadcast Domain
- VLANs allow a flat network to be divided into multiple smaller networks
- VLAN members can be connected anywhere in a campus network
- Switches are configured so that each port is mapped to a VLAN
- A Layer 3 device is required to enable communication between two or more VLANs

VLAN Membership
---------------

- Static VLAN

  * Port-based where port is manually assigned to a specific VLAN
  * No configuration required on the host
  * Port VLAN ID (PVID)
  * Hardware level switching via ASICs

- Dynamic VLAN

  * Ports assigned to VLAN based on the connected hosts MAC address
  * Requires external database hosted on a VLAN Membership Policy Server (VMPS)
  * Flexibiliy and Mobility
  * Greater administrative overhead

Static VLAN configuration
-------------------------

- VLAN Created with ID and Name
- VLAN 1 is default for every switch port
- Standard VLAN range 1-1005
- Extended VLAN range 1006-4094
- Extended VLAN Range can only be used in VTP version 3 or on VTP transparent switches
- Legacy VLANS 1002-1005 used for Token Ring and FDDI switching
- Name is optional, upto 32 characters with no spaces
- Switch port is configured as an "access" port when access to a single VLAN is required

Deploying VLANs
===============

- Cisco recommendes one-to-one relationship between vLAN and IP Subnet
- Should not allow VLAN to extend beyond Layer 2 domain of a distribution switch

  * Keep broadcasts out of core layer
  * VLAN stays within switch block
  * Limits Failure domain

- VLAN scaling methods

  * End-To-End VLANs
  * Local VLANs

End-To-End VLANs
----------------

- Campus wide, spans entire switch fabric
- Maximum Flexibility and Mobility
- Users not tied to physical location
- VLAN must be available at access layer of every switch block
- VLAN must be available on Core Layer
- Users should have same traffic flow patterns 
- 80/20 Rule - 80% traffic stays in workgroup, 20% remote
- Not recommended unless good reason
- Broadcasts are carried across entire switch fabric
- Broadcast storm or Layer 2 bridging issue could affect entire network

Local VLANS
-----------

- Traffic Flow should follow 20/80 Rule
- 20/80 Rule - 20% local traffic, 80% remote traffic
- Centralised Intranet/Internet Resources
- VLANs assigned around user communities based on Geographic boundaries
- Little regard for amount of traffic leaving VLAN
- Can be Implemented on single switch to an entire building
- Layer 3 functions handle Inter-VLAN traffic loads
- Maximum Availability and scaleability with redudant paths
- Small Failure Domain

VLAN Trunks
===========

- Trunk links transport traffic for one or more VLANs Over a single switch port
- Most used between a switch and other switches/routers
- Not assigne to a specific VLAN

VLAN Frame Identification
-------------------------

- Uses Frame "Tagging", added to each frame when carried over a trunk link
- "Tag" is removed when frame is sent over a non-trunking (Access) port
- Identification Methods

  * Inter-Switch Link (ISL) Protocol - Cisco Proprietary
  * IEEE 802.1Q - Standards Based

- Both ISL and IEEE 802.1Q increase frame size and can result in a frame 
  exceeding the maximum transmission unit (MTU). Referred to as "baby giants"

Inter-Switch Link Protocol (ISL)
--------------------------------

- Cisco Proprietary
- Original Frame Encapsulated Between ISL Header and Trailer, 30 bytes overhead

  * Header (26 bytes) contains 15-Bit VLAN ID (1-4094)
  * Trailer (4 bytes) contains CRC value for data integrity

- Only supported on higher end Cisco devices

IEEE 802.1Q Protocol
--------------------

- Standardised cross-vendor protocol
- Tagging information embedded into Layer 2 frame (single/internal tagging)
- Supports "Native" VLAN Where frames are sent over trunk link untagged
- 4-Byte tag added after source MAC Address in the original frame
- Tag contains 2 byte TPID, always has value 0x8100
- 2 bytes of TCI Field

  * 3 bit priority field for CoS
  * 12 bits for VLAN ID (VID

- VLAN IDs 0,1 andd 4095 are reserved
- Adds 4 bytes of overhead to each frame

Dynamic Trunking Protocol
=========================

- Cisco Proprietary
- Used to negotiate common trunking mode between switches
- Can negotiate if trunking is allowed and what protocol is used (either ISL or 802.1q)
- Must be used within same VTP domain or one or both switches have a null domain
- DTP frames are sent every 30 seconds
- ISL is preferred over 802.1Q if both devices support it
- Enabled by default (using "dynamic auto" mode) but only if requested by far end device
- DTP can be disabled on a per port based when not desired

Trunking modes
--------------

- Trunk - Port is permenantly trunking however DTP is stil operational
- Dynamic Desirable - Port actively tries to establish trunk with connected device
- Dynamic Auto - Port can form a trunk but only if far end requests it

Voice VLANs
===========

- Most Cisco IP phones contain an internal 3-port switch
- Link between IP phone upstream port and switch can negotiatiate a conditional trunk
- Conditional trunk allows for voice/data seperation and QoS prioritisation
- Voice packets are carried over the special "Voice VLAN" (VVID)
- The switch must be informed of the voice VLAN per-port
- DTP and CDP are used to negotiate trunk when needed

Support Voice VLAN Methods
--------------------------

- Specific VLAN ID - Trunk enabled, voice carried over vlan, data untagged
- dot1p - trunk enabled, VLAN 0 used for voice, data untagged
- untagged - Trunk enabled, voice and data untagged
- none - Default, no trunk, access VLAN used for both data and voice traffic

Wireless VLANs
=============

- Wireless Access Points (APs) provide connectivity etween wired and wireless devices
- APs Suports Autonomous and Lightweight operating  modes

Autonomous APs
--------------

- Independant operational
- Connects VLAN to WLAN one-to-one
- Requires a trunk link where multiple WLAN/VLAN mappings are used

Lightweight APs
---------------

- Cooperates with centralised Wireless LAN Controller (WLC)
- VLWN-WLAN trafffic encapulsated via a speciai tunnel to the WLC
- Tunnel uses "Control And Provisioning of Wireless Access Points" (CAPWAP) protocol
- Only needs access port configuration in order to communicate with WLC where loccal breakout is not used

VLAN Configuration Commands
===========================

**Create a VLAN**

::

  vlan <id>
    name <string>

**Assign a port to a single vlan (access port)**

::

  interface <name>
    switchport
    switchport access vlan <vlan-id>
    switchport mode access  

**List VLANs known to the switch and their assigned ports**

::

  show vlan [<id>] [brief]

**Configure a VLAN trunk**

::

  interface <name>
    switchport
    switchport trunk encapsulation {isl | dot1q |  negotiate}
    switchport trunk native vlan <id>
    switchport trunk allowed vlan {<vlan-list> | all | { add | except | remove } <vlan-list>}}
    switchport mode { trunk | dynamic {desirable | auto}}

**Disable/Enable DTP**

::

  interface <name>
    switchport trunk encapsulation {isl | dot1q}
    switchport mode {trunk | access}
    [no] switchport nonegotiate

**Verify Switch Port configuration and operational state**

::

  show interface <name> switchport


**Verify Trunking Information for a port**

::

  show interface <name> trunk

**Configure Voice VLAN**

*NOTE: Ensure VLAN has been created first*

::

  interface <name>
    switchport voice vlan {<id> | dot1p | untagged | none}


**Verify Voice VLAN is carried over the conditioanl trunk**

::

  show interface <name> switchport
  show spanning-tree interface <name>
