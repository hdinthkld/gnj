.. _cisco_transparent_fw:

###############################
Cisco ASA Transparent Firewalls
###############################

Overview
========

An ASA Firewall is capable of operating at Layer 2 when running in transparent
mode. This allows it to be installed into the network with minimal distruption
becaue no IP addressing changes are needed on the network.

This type of firewall is sometimes called a Layer 2 or "Stealth" Firewall as
it does not appear as a hop on the network and therefore is invisible to users,
a bump-in-the-wire.

Packets are forwarded from one interface on the ASA to another based on their
MAC adress. This requires the ASA to mantain a MAC address table so that it
knows which hosts exist on each of it's interfaces.

What differs the ASA from a switch is that whilst a switch will flood packets
for unknown packets out of all interfaces, the ASA instead will try to discover
the destination interface by the following methods:

* **ARP Request** when the destition IP is located on a directly connected
  subnet to the ASA.
* **Ping Request** when the destination IP adress is located on a distant
  subnet. This allows the ASA to learn either the next-hop routers MAC.

Advantages
----------

* Ability to filter traffic between hosts using higher-level protocols (e.g.
  IP addressing and ports) without readdressing the network.

* Can forward non-IP packets

Limitations
-----------

* No support for VPN transit traffic, management only.

* No QoS

* No Multicast

* No Dynamic routing protocols

* No DHCP relay

* No Dynamic DNS

* Unified Communications is not supported

Implementation
==============

Each of the ASAs interfaces need to be grouped into one or more bridge groups.
Each of these groups acts as an independent transparent firewall. It is not
possible for one bridge group to communicate with another bridge group without
assistance from an external router.

As of 8.4(1) upto 8 bridge groups are supported with 2-4 interface in each
group. Prior to this only oe bridge group was supported and only 2 interfaces.

When running in muliple context mode, each context can support multiple bridge
groups but each context must use a set of interfaces that is different from that
of another context.

All of the interfaces in the bridge group must share the same IP subnet. Packets
however are still inspected without the Layer 2 limitations which allows policy
decisions to be made on the full IP tuple (i.e. extended ACLs).

ASA versions 8.0(2) and above can also integrate with NAT when running in
transparent mode.

Whilst a firewall operating in routed mode can forward only IP packets, a
transparent firewall doesn't share this limitation. This requires a special
EtherType access-list.

In order to manage the firewall the dedicated management port must be used or
an IP address must be configured on one of the interfaces.

Configuration Steps
-------------------

#. Change the firewall mode
#. Configure interface groups
#. Assign IP address to the group
#. Create any management static routes
#. Configure Security Policies

.. rubric:: Change the firewall mode

This can only be done from the CLI, not through ASDM

.. code-block:: none

  firewall transparent

No reload is required after applying the command however the running
configuration will be cleared to remove any inapppriate settings.

.. rubric:: Configure interface groups

This can be configured via ASDM:

:menuselection:`Configuration --> Device Setup --> Interfaces`

Each bridge group is assigned a number.

From the CLI do the following for each interface:

.. code-block:: none

  interface <type/num>
    nameif <name>
    securty level <security-level>
    bridge group <number>
    no shutdown

.. rubric:: Assign IP address to bridge group

In ASDM it's necessary to create a new *Bridge Virtual Interface (BVI)*.

Through the CLI do the following:

.. code-block:: none

  interface BVI<number>
    ip address <ip> <subnet>

.. rubric:: Create management static routes

It the ASA needs access to any external networks (such as the Internet or
management networks), static routes need to be created (Dynamic routing not
supported).

Through the CLI this can be created as follows:

.. code-block:: none

  router <interface> <network> <mask> <gateway>


.. rubric:: Configure Security Policies

controlling IP traffic is configured in the same way as with a normal routed
firewall using *access-list* and *access-group* commands with the advantage
that non-IP trafffic (such as routing protocols, e.g. OSPF) can also be
permitted/denied.

It is also possible to configure rules for which the ASA does not have inbuilt
definitions by creating an EtherType rule.

In ASDM this done through:

:menuselection:`Configuration --> Firewall --> Ethertype Rules`

And the CLI:

.. code-block:: none

  access-list <name> ethertype [permit | deny] <ethertype>
  access-group <name> [in | out] interface <ifname>

This EtherType ACL will be applied together with any existing IP access list,
one of each type is allowing per inteface in each direction.

ARP Inspection and Spoofing
===========================

Because the ASA needs to learn which interface a given MAC address exists on,
an attacker could abuse ARP to leverage a man-in-the-middle attack.

Unlike higher-end switches the ASA cannot make use of the DHCP snooping table
but it is possible to configure the ASA with static ARP entries. This only
helps if the ARP requester and responder are located on different ASA
interfaces. If the ARP information received conflicts with the statically
configured entry, the ASA will assume the packet is spoofed and drop the packet.

The ASA can also be configured not to learn MAC addresses. This can however
become a high intesity management task as all MAC addresses would then have
to be configured statically.

In order to enable this feature:

#. Configure static ARP entries for all hosts
#. Enable ARP ispection
#. Disable MAC Address Learning (optional)

.. rubric:: Configure Static ARP entries

In ASDM this can be done via:

:menuselection:`Configuration --> Device Management --> Advanced --> ARP --> ARP Static Table`

Via the CLI:

.. code-block:: none

  arp <interface> <ip> <mac>

.. rubric:: Enable ARP inspection

ARP inspection is enabled on a per interface basis.  It is also possible to
configure the ASA to drop the packet if no ARP enty is found, the default
is to flood the packet so that it can be received.

Through ASDM ARP is enable via:

:menuselection:`Configuration --> Device Management --> Advanced > ARP > ARP Inspection`

And the CLI:

.. code-block:: none

  arp inspection <ifname> enable [ flood | no flood ]

.. rubric:: Disable MAC Address Learning

Stops the ASA from dynamically learning any MAC addresses as packets are
forwarded.  All MAC addressess must be statically defined.

To disable through ASDM on per-interface basis:

:menuselection:`Configuration --> Device Management --> Advanced --> Bridging --> MAC Learning`

Alternatively through through the CLI:

.. code-block:: none

  mac learn <interface> disable


Further Information
===================

For further reading please refer to the the Cisco CLI Guide on Transparent
Firewalls [c3]_.
