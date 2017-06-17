.. _cisco_dhcp_snooping:

*******************
Cisco DHCP Snooping
*******************

Overview
========

One of the problems that exists on all networks is how to ensure that a DHCP reply has
indeed come from an approved DHCP server, this helps defeat Man In The Middle attacks.
Outside of the security realm it can also assist with preventing users being able to plug
that random router they found at home into the network and use it as a switch...forgetting
that it was also running a DHCP server for their old broadband connection.

DHCP snooping works by tracking the communications between the end-user device and the
the DHCP server.  Any responses from untrusted DHCP servers are dropped.

Sources of DHCP information are defined as either Trusted or Untrusted.  This is done on
port level.  A port where DHCP replies should be seen, such as the uplink to a server switch
or access port connected to the DHCP should be marked as Trusted.  All user access ports
should be marked as untrusted.

Because the default trust state of all interfaces is untrusted, all trusted ports must be
manually configured.

The DHCP Snooping feature maintains a binding database, it contains an entry for each
untrusted host with a leased IP address if the host is associated with a VLAN that
has DHCP snooping enabled. It will not contain entries for hosts connected through trusted
interfaces. The binding maintains the MAC address of the host and leased IP address and the
bindings is maintained until the lease expires or the switch sees a DHCPRELEASE message from
the host.

The switch will forward the packet unless any of the following conditions match for packets
on an untrusted interface where the assigned VLAN has DHCP snooping enabled:

* The switch receives a DHCP packet from a DHCP server outside the network
* A packet is received on an untrusted interface where the source MAC and DHCP client hardware
  address do not match (only if MAC address verfication is enabled).
* The switch receives a DHCPRELEASE or DHCPDECLINE from an untrusted host with an
  entry in the DHCP snooping binding table and the information in the table does not
  match the interface on which the message received.
* The switch receives a DHCP packet that includes a relay agent IP which is not 0.0.0.0

An optional feature that can be enabled is MAC Address Verification. This feature verifies
that the source MAC address and the client hardware address in the DHCP packets on untrusted
ports match.

A number of other features (such as DAI) can make use of the DHCP snooping database in order
to secure the network further.


Implementation
==============

The following represents a minimal configuration with the following steps:

#. Ensure the DHCP server is operational
#. Ensure that the DHCP server is connected through a trusted interface
#. Enable DHCP snooping on at least one VLAN
#. Configure the DHCP snooping database agent
#. Enable DHCP snooping globally


.. rubric:: Configure the trusted interfaces where DHCP traffic should be seen

.. code-block:: none

  interface <type> <slot/num>
    ip dhcp snooping trust

.. rubric:: Enable DHCP snooping on a VLAN

.. code-block:: none

  ip dhcp snooping vlan <vlan-id>

.. rubric:: Enable DHCP snooping globally

.. code-block:: none

  ip dhcp snooping

  ! Optional
  ip dhcp snooping information option [replace | allow-untrusted ]

  ! Optional - Required for MAB operations with 802.1x
  ip dhcp snooping trust host

  ! Optional - Verify that the MAC address in the DHCP packet matches that in to Layer 2 Frame
  ip dhcp snooping verify mac-address

  ! Optional - Enable DHCP Server Detection
  ip dhcp snooping detect spurious vlan <vlan-id>
  ip dhcp snooping detect spurious interval <minutes>


.. rubric:: Configure the DHCP Snooping Database Agent

If the switch were to reboot and the DHCP database this could lead to network distruption.
It is recommended that the the database is stored on a TFTP server so that when the
switch reloads it will retrieve the latest database and reload the bindings without taking
unnecessary space on the switches flash memory.

.. code-block:: none

  ip dhcp snooping database <url>


.. rubric:: Add Static entries to the database

If a device with a static IP address is on a VLAN with DHCP snooping enabled, it needs to have
a static entry added otherwise frames may be dropped:

.. code-block:: none

  ip dhcp snooping binding <id> vlan <vlan-id> <ip> interface <type> <slot/num> expiry <lease_time>

Verification
============

To verify that DHCP Snooping is enabled use:

.. code-block:: none

  show ip dhcp snooping

  show ip dhcp snooping database

  show ip dhcpd snooping binding

  show ip dhcp snooping track host

  show ip dhcp snooping detect spurious


Troubleshooting
===============

To clear the DHCP snooping tracking cache:

.. code-block:: none

  clear ip dhcp snooping track host


To reload the DHCP snooping database form the specified URL :

.. code-block:: none

  renew ip dhcp snoop data <url>


External Reference
=================

**Cisco - Catalyst 6500 Release 12.SX Software Configuration Guide**

http://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst6500/ios/12-2SX/configuration/guide/book/snoodhcp.html

**Packet Pushers - Five things To Know About DHCP Snooping**

http://packetpushers.net/five-things-to-know-about-dhcp-snooping/
