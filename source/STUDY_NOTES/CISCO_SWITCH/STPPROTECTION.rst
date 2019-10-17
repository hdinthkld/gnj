***********************************
Cisco - Protecting the Spannning Tree Protocol Topology
***********************************

Overview
========

Cisco provides additional protections that can be put in place to prevent unauthorised or maliscious actions
from compromising the stability of the spanning tree topology.

This technologies include:

- Root Guard
- BPDU Guard
- BPDU Filtering
- Loop Guard
- Uni-Directional Link Detection (UDLD)

Root Guard
==========

- Controls whre candidate root bridges can be connected into the network
- Port will enter "Root-inconsistent" state if a superior BPDU is seen on a protected port
- No data can be sent/received until the superior BPDU's are no longer seen on the port
- Configured on a per port basis
- Use where the root bridge should never be expected (e.g. Access and Distribution layers)

Root Guard Configuration
------------------------

**Enable Root Guard**

::

  interface <name>
    spanning-tree guard root

BPDU Guard
==========

- Uses along with "PortFast" enabled ports
- Where a BPDU is detected on a protected port, port is put in an err-disabled state
- Disabled by default
- Can be enabled globally on all portfast enabled ports or on each port individually
- Port will remain err-disabled even after BPDUs stop, manual intervention needed to restore
- Does not protect against Bridging loops involving a hub as they do not send BPDU's

BPDU Guard Configuration
------------------------

**Enable BPDU Guard globally for all PortFast ports**

::

  [no] spanning-tree portfast bpduguard default

**Enable BPDU Guard on individual interfaces**

::

  interface <name>
  spanning-tree bpduguard {enable|disable}

BPDU Filtering
==============

- Effectively disables STP on the port
- Not usually recommended
- Prevents BPDUs being sent or processed on a port
- Can be enabled globally for all portfast ports or ion individual port basis

BPDU Filter Configuration
------------------------

**Enable BPDU Filter globally for all PortFast ports**

::

  [no] spanning-tree portfast bpdufilter default

**Enable BPDU Filter on individual interfaces**

::

  interface <name>
  spanning-tree bpdufilter {enable|disable}

Loop Guard
==========

- Protects against bridging loops when there is a sudden loss of BPDUs
- If a BPDU is seen on a port but then stops, port is moved to "loop-inconsistent" state
- Disabled by default 
- Enabled globally or per interface
- No manual intervention required
- Only the VLAN without BPDUS is affected
- Can safely be enabled on all switch ports

Loop Guard Configuration
------------------------

**Enable Loop Guard globally**

::

  [no] spanning-tree loopguard default

**Enable Loop Guard on individual interfaces**

::

  interface <name>
    [no] spanning-tree guard loop

.. _switch_udld:

Uni-Directional Link Detection (UDLD)
=====================================

- Protects against BPDUS only being received in one direction
- Cisco proprietary feature
- Uses an Echo and Reply process
- The switches at either side of the link must have UDLD enabled
- Default interval is 15 seconds, failure assumes after 3 times this interval
- Enabled globally or on per port basis
- No action taken until the first successful two-way communication is completed
- Can operate in "Normal" or "Aggressive" moved
- Normal mode only sends an alert, the port remains active
- Aggressive mode sends a message every 8 seconds to the other switch upon failure detection, if
  no further response port is put in err-disabled state
  
UDLD Configuration
------------------------

**Enable UDLD globally for all fibre optic ports**

::

  [no] udld {enable | aggressiive | message time <seconds>}

**Enable UDLD on individual interfaces**

::

  interface <name>
    udld {enable | aggressive | disable }

**Renable all ports disabbled due to UDLD**

::

  udld reset

Troubleshooting STP protection
==============================

::

  show spanning-tree inconsistent ports
  show spanning-tree interface <name> [detail]
  show spanning-tree summary
  show udld