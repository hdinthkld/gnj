***********************************
Cisco - Preventing Snooping Attacks
***********************************

.. _switch_snooping_dhcp:

DHCP Snooping
=============

- Mitigates the risk of a rogue DHCP server existing on the network
- Ports configured as trusted or untrusted
- All DHCP requests are intercepted by the switch
- DHCP replies on untrusted ports are discarded and source port is shutdown
- Switch creates database of DHCP "Bindings" as legitimate replies received

- Configuration Steps

  - Define Trusted Ports
  - Enable DHCP Snooping
  - Define What VLANs to enable DHCP snooping on

.. _switch_ipsourceguard:

IP Source Guard
===============

- Mitigates the risk of maliscious hosts spoofing their source IP address
- Takes advantage of MAC-To-IP mappings stored in the DHCP snooping database
- Static entries can also be configured for non-DHCP hosts
- Packets must pass conditions in order to pass thhrough switch
  
  * Source IP identical to IP learned by DHCP snooping/Static entry
  * Source MAC identical to MAC learned by switch and DHCP Snooping/Static entry

- Dynamic ACL or port security is used to filter traffic

- Configuration Steps
  
  * Enable DHCP Snooping
  * Enable Port Security (if detecting MAC Spoofing is required)
  * Configure Static Entries for non-DHCP hosts
  * Enable Source Guard (Per Interface)

.. _switch_dai:

Dynamic ARP Inspection (DAI)
============================

- Mitigates the risk of ARP poisoning/ARP spoofing
- Uses either static entries or those learned through DHCP snooping
- Ports classed as trusted/untrusted
- Ports connecting other switches should be trusted
- Switches checks ARP replies received on untrusted port
- Invalid ARP replies are dropped and logged

- Configuration Steps

  * Define trusted interfaces
  * Confiigure ARP access-list for non-DHCP hosts
  * Enable DAI on required VLANs

- MAC Access-list (Filter is checked first then the DHCP snooping database
- Checking of DHCP snooping database can be skipped if required

- Only MAC/IP in the ARP reply are checked, additional ethernet level checks can be enabled

  * Source MAC
  * Destination MAC
  * IP

Configure DHCP Snooping
=======================

**Enable DHCP Snooping**

::

  ip dhcp snooping

**Specify VLANs to have DHCP Snooping applied**

::

  ip dhcp snooping vlan <start-vlan-id> [<end-vlan-id>]

**Specify Trusted Interfaces**

::

  interface <name>
    [no] ip dhcp snooping trust

**Limit rate of DHCP requests**

::

  interface <name>
    [no] ip dhcp snooping limit rate <pps>

**Add option 82 to DHCP request**

*Note: Enabled By Default*

::

  [no] ip dhcp snooping information option

**Verify DHCP Snooping**

::

  show ip dhcp snooping [binding]

Configuring IP Source Guard
===========================

**Configure Static Binding**

::

  ip source binding <mac> vlan <id> <ip> interface <name>

**Enable IP Source Guard on an interface**

::
  
  interface <name>
    ip verify source [port-security]

**Verify IP source Guard Status**

::

  show ip verify source [interface <name>]

**Verify Source Bindings**

::

  show ip source binding [<ip>] [<mac>] [dhcp-snooping|static] [interface <name>] [vlan <id>]

Configuring Dynamic ARP Inspection (DAI)
========================================

**Configure Trusted Interfaces**

::

  interface <name>
    ip arp inspection trust

**Enable DAI on required VLANs**

::

  ip arp inspection vlan <vlan-range>

**Define MAC Access List foor non-DHCP hosts**

::

  arp access-list <arp-acl-name>
    permit ip host <ip> mac host <mac> [log]

**Apply Filter**

::

  ip arp inspection filter <arp-acl-name> vlan <vlan-range> [static]

**Apply additional packet validations**

::

  ip arp inspection validate {[src-mac] [dst-mac] [ip]}

**Display DAI Status**

::

  show ip arp inspection

  