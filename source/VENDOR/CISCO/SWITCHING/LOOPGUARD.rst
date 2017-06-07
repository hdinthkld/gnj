################
Cisco Loop Guard
################

Overview
--------

Spanning Tree is used to resolve a loop-free network topology where there may be multiple
redudant links.  It does this by forming a tree-like struction and removes/disables
sub-optimal branches so that the structure is loop-free.

It does this by sending out BPDUs and waiting for other switches to send their own
BPDUs.  After a certain time period the Root Bridge is selected based on the preferred
(lowest Bridge Priority or Bridge ID in a tie). All subsequent calcuations are based of
the location of the root bridge in the network.

The problem with this is that some loops may not be detected if BPDUs are not received
through certain ports.  For example due to hardware (such as faulty cables) or software
(buggy software), some devices (such as Hubs) do not even process BPDUs so will blindly send
them on to other devices or via another port on the same device.

Cisco developered a feature known as LoopGuard which is intended to assist with this issue.

LoopGuard takes additional steps to ensure that BPDU are being received from other
devices when it is enabled on non-designated ports.  If BPDUs are not received on the port
LoopGuard assumes there is a problem and puts the port into a loop-inconsistent state.

This may seem counter productive but if a port is a designated port it should be sending
BPDUs so the lack of them indicates an issue.

When a port is put into the loop-inconsistent state, a syslog message will be generated:

*%SPANTREE-2-LOOPGUARD_BLOCK: Loop guard blocking port <type> <slot/num> on VLAN<vlan-id>.*

The presence of a BPDU is enough for the port to recover so no manual intervention is
required.


Interoperability with Other Switch Security Features
====================================================


.. rubric:: Root Guard


Loop Guard and Root Guard cannot be deployed together.  This makes sense as Root Guard
is deployed on Designated Ports where as Loop Guard works on Non-Designated Ports.

If Loop Guard is configured, Root Guard is automaticaly disabled on the Port.

.. rubric:: PortFast / BPDU Guard


LoopGuard cannot be enabled on ports with BPDUGuard.

It should also be noted that LoopGuard cannot be enabled on dynamic VLAN ports.


.. rubric:: Uplink Fast / Backbone Fast

Uplink and Backbone Fast are transparent to LoopGuard so they do not trigger LoopGuard
in any way.


LoopGuard and UDLD
==================

LoopGuard shares much of its functionality with another feature known as Uni-Directional
Link Detection (UDLD) however whilst LoopGuard is intended to defend against STP Loops
UDLD is a more general feature to defend against a number of hardware (e.g. wiring errors)
issues. The changes of a uni-directional link are considered more likely than that of
an STP issue and UDLD also provides benefits where EtherChannel is used as only a single
interface would be disabled, not the entire EtherChannel. 

LoopGuard and UDLD can be enabled together.


Implementation
---------------

Originally Loop Guard could only be enabled on a per-port basis, now it is supported to be
enabled globally.  If this is done it will be enabled on all point-to-point links (all
ports that are full-duplex).

To enable globally use the following command:

.. code-block:: none

  spanning-tree loopguard default

To enable per-interface use:

.. code-block:: none

  interface <type> <slot/num>
    spanning-tree guard loop


Verifying
---------

To verify if Loop Guard is enabled use:

.. code-block:: none

  show spanning-tree summary


Troubleshooting
---------------

When a port is put into the loop-inconsistent state, a syslog message will be generated:

*%SPANTREE-2-LOOPGUARD_BLOCK: Loop guard blocking port <type> <slot/num> on VLAN<vlan-id>.*

If these messages are logged repetatively for the same port it should be investigated as
it could be an indication of a faulty cable or device.



