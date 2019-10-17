***********************************************
Cisco - Using Port Mirroring To Monitor Traffic
***********************************************

Port Mirroring Overview
=======================

- Switched Port Analyzer (SPAN)
- Allowing a network analyzer to be attached to a switch receive all packets irrespetive of any MAC learning or VLAN
- Packets received on source port are copied to the destination port

SPAN Types
----------

- Local Span - Both Source and Destination ports on the same switch
- Remote Span - Source and Destination ports on different switches

.. _switch_span_local:

Local SPAN
==========

- Span session is setup by specifying both a source and destination
- Source and destination must be on the same switch
- Session can be configured to mirror received, transmitted or both directions of traffic to the destination 
- Source can be a single port, EtherChannel or Vlan
- Destination must be a single port
- Source and destinations should have equivilent traffic handling capabilities otherwise packets could be lost
- STP is disabled on the destination port
- Any traffic that is received on the destination port is by default dropped


Local SPAN Configuration
------------------------

**Define SPAN Source**

::

  monitor session <number> source {interface <name> | vlan <id> [rx|tx|both]

**Define SPAN destination**

::

  monitor session <number> destination interface <name>
          [encapsulation replicate]  <---------------------- Use to mirror Layer 2 protocols as well
          [ingress {dot1q vlan <id> |isl|untagged vlan <id>}]

**Filter VLANs to capture from Trunk Source**

::

  monitor session <number> filter vlan <vlan-range>

.. _switch_span_remote:

Remote Span
===========

- Enables the traffic analyzer to be located in a different part of the campus network to the source device
- Uses a special VLAN marked for Remote SPAN use
- If the source and destination switches are not directly connected, each switch along the path must know of the RSPAN VLAN
- It is recommended to remove the RSPAN VLAN from all trunks except those in the path
- VTP will correctly propagate the RSPAN VLAN and auto prune (if enabled) from unnecessary links
- Use One RSPAN VLAN for each each RSPAN session
- STP must run on the RSPAN VLAN therefore BPDUS can't be monitored

Remote SPAN Configuration
-------------------------

**Create RPAN VLAN on all potential switches in the path**

::

  vlan <id>
     name <name>
     remote-span

**Configure RSPAN Source Device**

*NOTE: Ensure RSPAN VLAN has been created first**

::

  monitor session <number> source {interface <name> | vlan <id>} [rx|tx|both]
  monitor session <number destination remote vlan <id>

** Configure RSPAN Destination Device**

*NOTE: Ensure RSPAN VLAN has been created first**

::

  monitor session <number> source remote vlan <id>
  monitor session <number> destination interface <name>
                  [encapsulation replicate]
                  [ingress {dot1q vlan <id>|isl|untagged vlan <id>}]

Managing SPAN Session
======================

**Show Configured Sessions**

::

  show running-configuration | include monitor
  show monitor [session {<number> | all | local | range <list> | remote} [detail]

**Delete SPAN Session**

::

  no monitor session {<session> | range <session-range>} | local | all}
