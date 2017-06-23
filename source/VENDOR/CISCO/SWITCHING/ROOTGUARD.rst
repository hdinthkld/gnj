.. _cisco_rootguard:

################
Cisco Root Guard
################

Overview
--------

In standard Spanning Tree there is no means by which to securely enforce the switched
Layer 2 Topology. When spanning tree calculates the topology of the network it is based on
the location of the root bridge and any switch with the lowest bridge ID will traditionally
take on this role. Without some additional controls the administrator cannot enforce the
position of the root bridge.

Cisco has added an enhancement called Root Guard to the Spanning Tree Protocol which gives
administrators a means by which to say where the root bridge is expected to be. This feature
ensures that the port on which root guard is enabled is the designated port. Normally root
bridge ports are all designated ports.

The feature works by detecting superior STP BPDUs on a root guard-enabled port. When such a
BPDU is detected the port upon which the BPDU is seen is put in the root-inconsistent state
and not allowed to communicate any further on the network.

This feature is useful for protecting against a rogue switch being deployed on the network
and attempting to become the root bridge. If such a switch is connected to a port of a
switch under the administrators control and it has root guard enabled, the port will be
put back in the listening state effectively preventing it communicating its BPDUs onto the
network.

Once the port with Root Guard enabled stops receiving BPDUs, it will unblock the port. No
manual intervention is required. Each time the port is blocked a syslog entry will be logged
so that monitoring and alerting systems can detect the event:

*%SPANTREE-2-ROOTGUARDBLOCK: <type> <slot/num> tried to become non-designated in VLAN <vlan-id>. Moved to root-inconsistent state*

Implementation
--------------

Root Guard is enabled on a per port basis and should be configured on all ports where
superior BPDUs are not expected to be seen.  An example of this would be all the ports where
end user devices are connected and those that connect to unmanaged/3rd party switches. It
can be imagined as setting up a perimeter around the network where the STP root should be
located.

It should be obvious however to be clear, do not configured Root Guard on the ports facing
the root bridge.


Enabling the feature is a simple one line command that needs to be applied to all ports where
the root bridge will not be seen:

.. code-block:: none

  interface <type> <slot/num>
    spanning-tree rootguard


When implementing care must be taken to ensure a proper understanding of the network topology.
This becomes more difficult when two potential root bridges are used for high availability
(such as dual Cisco 3750 stacks or a pair of Cisco 6500 switches). A well designed network
topologies that follow best practice (such as the Hierarchical Switch Design from Cisco) can
make it easier to identify where Root Guard should be deployed.

In a basic sense ensure that the root BPDUs never have to traverse a non-managed switch or
user enabled port. Ports between Access, Distribution and Core devices switches should be
the only places where Root Guard is not deployed unless there is good reason not to.

Verification
------------

Troubleshooting
---------------

The status of a port can be determined by:

.. code-block:: none

  show spanning-tree vlan <vlan-id>

Any ports that have been blocked by Root Guard will show as 'ROOT_Inc'

External Reference
------------------

**Cisco - Spanning Tree Protocol Root Guard Enhancement**

http://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/10588-74.html

**Roger Perkin - Spanning Tree Root Guard**

http://www.rogerperkin.co.uk/spanning-tree/spanning-tree-root-guard/
