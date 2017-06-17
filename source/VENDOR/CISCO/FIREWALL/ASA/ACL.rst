.. _cisco_asafw_rules:

=====================================
Cisco ASA Firewall - Rules Management
=====================================

Overview
========

The Cisco ASA is a dedicated firewall appliance and has much more structure
to the way in which traffic filtering is applied that a general purpose
router firewall.

Unlike a router the filtering of traffic to the firewall is handled seperately
than transit traffic through the device, so there is no risk of loosing
management access when amending user policies.

.. _cisco_asa_unified_acl:

Unified ACL
===========

Prior to version 9.0(1) it was necessary to manage IPv6 and IPV4 access
lists seperately.  Later versions have unified this functionality and
removed the IPv6 specific commands.

This now means that a single access-list can now include a mixture of
both IPv4 and IPV6 hosts and networks. Object groups can also be include a
mixture of these addresses as well befoe being bound to the ACL.

In order to support this the 'any' keyword has been modified to include
both IPv4 and IPv6 addressing.  The new 'any4' and 'any6' keywords now refer
to their respective IP addressing version (IPv4 and IPv6 respectively).

Although the new features allow the specifying of an IPv4 and and IPv6
destination (and vice versa), communication between these devices will only
work if NAT46/NAT64 has been configured to map the addresses to a valid 
address in the other IP version.  Without NAT46/NAT64 the commands are still
valid but only communication betwee the same IP versions will function
correctly.

Implementation
==============

In order to implementing filtering on the firewall the following steps must
be done:

.. rubric:: Necessary Steps

#. Create Host, Network or Service objects
#. Create Network/Service Object Groups
#. Combine raw IP addresses, subnets and/or objects/object groups into an ACL
#. Apply the ACL to an interface either inbound or outbound.


.. rubric:: Create Host, Network or Service Objects

Objects are used on the firewall in order to reduce duplication and enable
easier management of the firewall in general.

An object can be used to represent a single host or network as well as
a specifically named service or protocol.

.. note:: The ability to create named objects on the firewall was added in
          ASA version 8.3.  Prior to this it was necessary to use *name*
          statements but this only supported hosts, not networks or services.

To define needed objects, the following syntax applies for Network objects:

.. code-block:: none

  object network <net-obj-name>
    [description <text>]
    {host <ip> | subnet <ip> <mask}

And for Service objects:

.. code-block:: none

  object service <svc-obj-name>
    [description <text>]
    service <protocol> {source | destination} <port>


.. rubric:: Define Object Groups

Object Groups are used to allow either a group of hosts/networks or services
to be combinded into a single configuration item.  This can then be added to
an ACL and allows the specified group to be used multiple times, helping to make
the firewall policy more managable.

For a network group:

.. code-block:: none

  object-group network <net-objgrp-name>
    [description <text>]

    ! Add a previously configured  network object group
    group-object <net-objgrp-name>

    ! Add an existing object to group
    network-object object <net-obj-name>

    ! Use a raw IP address
    network-object host <ip>

    ! Use a raw subnet
    network-object <ip> <mask>


.. note:: Ability to use object groups was added in ASA version 7.0(1). Ability
          to use mixed IPv4 and IPv6 was added in a single group was added in
          9.0(1)


.. rubric:: Define ACL

Once all the objects are defined an ACL an then be created from them:

.. code-block:: none

  access-list <acl-name> [line <line-no>] remark <text>
  access-list <acl-name> [line <line-no>] {pemit | deny } {protocol} <src-ip-spec> <dst-ip-spec> [<port-spec>]

.. rubric:: Bind ACL to interface

Each interface can have a unique ACL in the inbound and outbound direction.
Inbound ACLs are more common but outbound is applied in some special cases.

.. code-block:: none

  access-group <acl-name> {in | out} interface <ifname>


It is also possible to use a global ACL  which applies to all interfaces.
The global access list is applied after the interface specific ACL.

.. code-block:: none

  access-group <acl-name> global
