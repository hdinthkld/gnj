**********************
Cisco - Securing VLANs
**********************

VLAN Access Lists (VACLs)
=========================

- Normal router access lists (RACLs) only filter traffic between VLANs
- VACLS filer traffic that stays within a VLAN
- VACLs are merged into the TCAM
- Support Additional "Redirect" action
- Configured globally for the VLAN not per interface
- Evaluated in sequence
- Can match IP addresses as well as MAC addresses however they need to be in different ACLs
- Action and Match is performed as frammes within a VLAN or routed in/out of a VLAN
- Packets are filtered in hardware
- The "Redirect" action causes traffic to be forwarded onto a specified interface

.. _switch_vlan_private:

Private VLANs
=============

- Allows for segmenting of hosts within a VLAN without needing to assign ACLs or different subnets
- Hosts are limited to being able to talk to hosts on the "Primary" VLAN (e.g. a router) but not 
  hosts on a different "Secondary" VLAN
- Secondary VLANs must be mapped to one Primary VLAN
- VTP does not pass information relating to private VLANs, making them locally significant only
- Each switch port must be configuered to be either a "Promiscuous" or "Host" Port

Secondary VLAN Types
--------------------

- Isolated
  
  * Can only communicate with the primary VLAN
  * Cannot communicate with hosts on the same or different secondary VLANs

- Community

  * Can commmunity with primary VLAN
  * Can communicate with hosts on same Secondary VLAN
  * Cannot communicate with hosts on different Secondary VLAN

Port Modes
----------

- Promiscuous - Can communicate with any other host
- Host - Only communicate with same community or promiscuous port

Configuration Steps
-------------------

- Create the Private VLANs and specify the secondary VLAN type
- Create Primary VLANs and associate with secondary VLANs
- Set the secondary VLAN mode per interface
- Associate host ports with primary and secondary vlan
- Associate promiscuous ports with the primary and one or more secondary VLANs
- MAP SVI interfaces to Secondary VLANs

Securing VLAN Trunks
====================

Switch Spoofing
---------------

- Don't use DTP
- Manually configure expected behaviour on each port
- Prevent a maliscous host fromm forming a trunk

VLAN Hopping
------------
- Attacker will send double tagged frames with the outer tag as the native VLAN
- Switch will see the outer tag as being the same as the native VLAN and remove it
- Switch will now sent the frame onto the trunk with the previously hidden inner VLAN
- Attacker has now gained access to the other VLAN

- Conditions required to work

  * Attacker connected to access port
  * Same switch must have an 802.1q Trunk
  * Trunk must have attacker access VLAN as it's native VLAN

- Solution

  - Set Native VLAN to an unused VLAN ID
  - Prune the Native VLAN off both ends of the trunk


Configuring VLAN Access Lists
=============================

**Configure the VACL along with conditions**

::

  vacl access-map <name> [<seq-no>]
    match ip address {<acl-name> | <acl-no>}
    match mac address <acl-name>
    action {drop | forward [capture] | redirect <inteface-name>}

**Apply the VACL to one or more VLANS**

::

  vlan filter <name> vlan-list <vlan-list>

Configure Private VLANs
======================

**Create Secondary VLAN**

::

  vlan <id>
    private-vlan {isolated | community}

**Create Primary VLAN and Map to Secondary VLANs**

::

  vlan <id>
    private-vlan primary
    private-vlan association {<secondary-vlan-list> | add <vlan-list> | remove <vlan-list>}

**Set the Interface Port Modes**

::

  interface <name>
    switchport mode private-vlan {host|promiscuous}

**Associate Host Port With Primary And Secondary VLAN**

::

  interface <name>
    switchport private-vlan host association <primary> <secondary>

**Associate Promiscuous Ports with Primary and one or more Secondary VLANs**

::

  interface <name>
    switchport private-vlan mapping <primary> {<vlan-list>|add <vlan-list>|remove <vlan-list>}

**Associate Layer 3 SVI with one or more secondary VLANs**

::

  interface vlan<id>
    private-vlan mappiing {<vlan-id>|add <vlan-list>|remove <vlan-list>}

Vlan Trunk Secure Configuration
===============================

**Statically Configure An Interface as an access port**

::

  interface <name>
    switchport mode access
    switchport access vlan <id>

**Change the  Native VLAN of Trunk and remove from the trunk**

::

  interface <name>
    switchport trunk native vlan <id>
    switchport trunk allowed vlan remove <id>

**Specify the tag should be added even for native VLAN**

*NOTE: This is a global setting, not per interface*

::

  vlan dot1q tag native


