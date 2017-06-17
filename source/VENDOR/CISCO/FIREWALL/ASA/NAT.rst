.. _cisco_asa_nat:

=======================================
Cisco ASA - Network Address Translation
=======================================

Overview
========

NAT Order of Operations
-----------------------

.. rubric:: Pre-8.3

#. NAT Excemption
#. Static NAT
#. Static PAT
#. Policy NAT/PAT
#. Identity NAT
#. Dynamic NAT
#. Dynamic PAT

.. rubric:: Version 8.3 and Later

#. Manual NAT (Section 1)
#. Auto NAT (Section 2)
#. Manual NAT (Section 3) with *after-auto*


NAT Control
-----------

In versions of ASA prior to 8.3 a feature existed called NAT Control which
caused any traffic that did not have a corresponding NAT rule to be dropped.

It could be enabled/disabled as follows:

.. code-block:: none

  [no] nat-control

This feature is no longer available in later ASA versions.

Static NAT/PAT
--------------

Static NAT/PAT is useful when you want to have a host or group of hosts to
always appear as the same IP address on another network.  this provides a
one-to-one mapping between the real and translated address.

Most commonly these feature is used  where you need to provide a service to
users and need to be able to provide them with consistennt information.

The primary difference with Static NAT/PAT is that whilst both can translate the
IP address, PAT (or Port Redirection) can also translate the port number.  This
can be useful when you want to conserve IP addresses and use a single IP address
for multiple services where each service (such as TCP 25, TCP 443, TCP 53) are
all directed to different hosts on the internal network.

.. rubric:: Pre-8.3 configuration

.. rubric:: Post-8.3 configuation

Dynamic NAT/PAT
---------------

As the name suggests Dynamic NAT/PAT does  not provide a real host IP with a
consistent one-to-one mapping.  Instead when a connection is needed from a
host the ASA wil dynamically assign an IP address out of a pool of addresses
based on availability.  In the case of Dynamic PAT the source ports will also
potentially be modified which allows for the potential of an entire network
to be hidden behind a single public IP address (up to 65535 translations). ASA
8.4(3) and above extend this by allowing up to 65525 translations per service,
not address.

.. rubric:: Example of post-8.3 Dynamic PAT

.. code-block:: none

  object-network <obj-name>
    host <ip>
    nat (real-interface, translated-interface) interface service <protocol> <service> <service>

Policy NAT/PAT
--------------

Where as Static and Dynamic NAT/PAT apply to all connections, a Policy NAT can
be made to only be used when certain criteria is met (such as to only specific
destinations).

Identity NAT
------------

Identity NAT is used where you don't want translation to occur. This may be
necessary where you already have a Dynamic  NAT rule in place for general
Internet access but need to exclude traffic across a VPN from translation.
