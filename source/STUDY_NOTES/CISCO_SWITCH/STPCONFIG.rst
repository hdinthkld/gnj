***********************************
Cisco - Spanning Tree Configuration
***********************************

Spanning-Tree Global Configuration
==================================

**Disable/Enable Spanning Tree Protocol On a VLAN**

*Note: Can be used either globally or on a per-port basis*

::

  [no] spanning-tree vlan <id>

**Configure Bridge ID Format**

- Traditional 802.1D (STP) Uses a unique MAC address
- 802.1t uses a non-unique MAC address with bridge ID plus VLAN ID
- 802.1t is the default where a switch cannot suport 1024 uniquq MAC adresses
- Disabling "extended system ID" reverts to 802.1D bridge id format (if supported)

::

  [no] spanning-tree extend system-id

**Manually set Bridge ID format**

*Note: Must be uset in increments of 4096

::

  spanning-tree vlan <vlan-list> priority <value>

**Automatically set primary/secondary root bridge with macro

* Note: command will not show in the config directly

::

  spanning-tree vlan <id> root { primary | secondary } [diameter <value>]


**Tuning Port ID/Priority**

*Note: Port number is fixed, only priority can be changed

::

  interface <name>
    spanning-tree vlan <vlan-list> port-priority <value>

** Tune global STP timers

::

  spanning-tree [vlan <id>] hello-time <seconds> # 2 seconds by default
  spanning-tree [vlan <id>] forward-time <seconds> # 15 seconds by default
  spanning-tree [vlan <id>] max-age <seconds> # 20 seconds by default

Verifying Spanning Tree Operation
=================================

**Display STP Bridge Priorities**

::

  show spanning-tree [vlan <id>]

**Verify port cost**

::

  show spanning-tree interface <name> [cost]

**Monitoring STP**

::

  show spanning-tree [detail | summary]
  show spanning-tree [vlan <id>] root
  show spanning-tree [vlan <id>] bridge
  show spanning-tree interface <name>

PortFast
========

**Disable/Enable Portfast global for all non-trunk ports**

::

  [no] spanning-tree portfast default

**Disable/Enable PortFast on specific interfaces**

::

  interface <name>
    [no] spanning-tree portfast

**Automatically apply optimal configuration for an end user port**

*Note: Enables PortFast, sets to be an access port, disables PAgP

::

  interface <name>
    switchport host

**Verify PortFast status**

::

  show spanning-tree interface <name> portfast

UplinkFast
==========

**Enable UplinkFast globally**

::

  spanning-tree uplinkfast [max-update-rate <pps>]

**Verify UplinkFast Status**

::

  show spanning-tree uplinkfast

BackboneFast
============

**Enable BackboneFast Globally**

::

  spanning-tree backbonefast

**Verify BackboneFast Status**

::

  show spanning-tree backbonefast