.. _cisco_iosfw:

==================
Cisco IOS Firewall
==================

Overview
--------

The Cisco IOS firewall offers basic stateless firewall features. This type
of firewall should be used only as a last resort where no other options are
available.

Limited tracking of established connections is available to avoid having to
specifically allow all inbound traffic.

The functional components of the IOS firewall are:

* ACL specifying the traffic to allow or deny
* Internet to ACL binding via an access group

The same set of commands is available for IPV6 but instead of *ip* use
*ipv6* as the prefix.

Implementation
--------------

.. rubric:: Steps Necessary
#. Create any object groups necessary
#. Define the Access Control List that meets the security requirements
#. Bind the Access Control List to the relevant interface, inbound and/or
   outbound.

.. rubric:: Create Object Groups

Object groups can be either for specifying groups of IP addresses (subnets or
hosts) and/or for specifying groups of service ports.

.. note:: This feature is available since IOS 12.4(20)T

.. code-block:: none

  object-group network <net-group-name>
    network-object {<ip> <subnet> | host <ip>}

  object-group service <svc-group-name>
    <protocol> <port-or-range>

It is also possible to nest groups within another group. The group must
exist before being included in another one:

.. code-block:: none

  object-group network <net-group-name-2>
    group-object <net-group-name-1>

  object-group service <svc-group-name-2>
    group-object <svc-group-name-1>

.. rubric:: Define the ACL

The ACL is where all the source/destination IPs and port numbers are combined so
that the policy can be defined.

Prior to IOS 12.4 it was necessary to recreate the entire ACL when any edits
needed to be made other than at the end, this was done through the
*access-list* command.

Since 12.4 however the "ip access-list" command is
preferred as it allows line numbers to be specified so rules can be inserted
anywhere there is a gap in the numbering.  It is recommended to leave gaps (e.g.
in intervals fo 10) to allow for further changes.

Here is example of the pre-12.4 way of implementing an ACL:

.. code-block:: none

  access-list <name-or-number> extended {permit | deny} <protocol> <source-net> <source-mask> <dst-net> <dst-mask> [eq <dst-port>] [log]


And the newer style post-12.4:

.. code-block:: none

  ip access-list extended <name>
    [<line-no>] {permit | deny} <protocol> <source-net> <source-mask> <dst-net> <dst-mask> [eq <dst-port>] [log]

For TCP based connections the *established* keyword can also be specified to
permit established connections back through the firewall.


It is also possible to renumber the ACL using the following command:

.. code-block:: none

  ip access-list resequence <name-or-number> <start-no> <interval>


.. rubric:: Bind the ACL to an interface

Once created the ACL must be bound to the appropriate interface.  This can
be done either inbound or outbound:

.. warning:: When applying ACL to the interface through which the device
             is being monitored, it is important that the management traffic
             also be permitted otherwise the session will be disconnected.

.. code-block:: none

  interface <type> <slot/num>
    ip access-group <name-or-num> {in | out}


Verifying
---------

The following commands can be used to verify the status of an access list:

To see the hits against the various rules use:

.. code-block:: none

  show access-list

To see what access list is bound to a specific interface use:

.. code-block:: none

  show ip interface <type> <slot/num>

Troubleshooting
---------------

If you need to verify what packets are being denied or allowed, it is possible
to log the packets being matched by using the *log* argument on the individual
ACE.

Note that this does put additional burden on the device so should not be done
for long periods of time.
