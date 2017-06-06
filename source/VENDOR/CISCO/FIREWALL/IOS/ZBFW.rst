.. _cisco_zbfw:

===========================
Cisco Zone Based  Firewall
===========================

Overview
--------

Cisco Zone-based Firewall or ZBFW for short is an updated means of providing
firewall features in a WAN environment.

The benefits of ZBFW over the Legacy IOS Firewall (known as Context-Based Access
Control or CBAC) include:

* Stateful Packet Inspection (no more need for established keyword)
* VRF-aware
* URL filtering (basic)
* Denial-Of-Service (DoS) mitigation
* Application Inspection

.. note:: Initial zone based firewall features were added in 12.4(6)T with the
          newer application inspection added in 12.4(9)T

A number of IOS Firewall features are however not supported:

* Authentication Proxy
* Statefull Firewall Failover
* Unified Firewall MIB
* IPv6 Stateful Inspection
* TCP out-of-order support

.. note:: The above limitations is correct up to 12.4(15)T

Implementation
--------------

In order to implement ZBFW it is necessary to understand where the security
boundaries (zones) exist as far as the router interfaces are concerned.

Each interface can only be in a single zone however by default, all interfaces
in the same zone can communicate.

.. note:: Unlike with the Legacy IOS CBAC Firewall, applying filtering to
          the interface through which the device is being monitored, will
          not result in a loss of connectivity.  A pre-defined zone called
          *self* exists on the router and traffic to this must be explicitly
          denied in order for traffic to the router to be blocked, this is
          the opposite of transit traffic through the router.

.. rubric:: Configuration Steps

#. Define matching rules for policy traffic (*class-map*)
#. Apply actions to policy traffic (*policy-map*)
#. Define zones (*zone*)
#. Define zone pairs (*zone-pair*)
#. Assign interfaces to zones  (*zone-member*)

.. rubric:: Define matching rules for policy traffic

ZBFW uses the same command set as also used with QoS, namely the Modular QoS
CLI.  It is composed a number of commands that allow traffic to matched either
by an access-list or known protocol (such as TCP or HTTP).

This class-maps are used for application inspection. For example:

.. code-block:: none

  class-map type inspect [match-any | match-all] <cm-name>
    match protocol <protocol-name>
    match access-group <acl-name-or-number>

If the protocol is known then the state of the connections will be tracked.

.. rubric:: Apply Actions to policy traffic

Once the traffic that needs to be either permitted or denied is known, it is
necessary to apply the action to it.

The action can either be:

* **Drop** - Traffic is denied
* **Pass** - Allow traffic but do not track the state of the connection
* **Inspect** - Allow traffic and provide stateful firewall tracking to the connection

The policy is applied as follows:

.. code-block:: none

  policy-map type inspect <pm-name>

    ! Specify the traffic class(es) created in the previous step
    class type inpect <cm-name>
      {inspect | drop | pass}


.. rubric:: Define Zones

A zone defines a area of the network which has a different security requirement
that other zones.  For example a router may be configured with a trusted,
untrusted and DMZ zone.

A zone is created as follows, all zones must have unique names:

.. code-block:: none

  zone security <zone-name>

.. rubric:: Define zone Pairs

A pair of zones defines a location where policy must be implement, if traffic
is to be allowed to be initiated in both directions two zone pairs are needed
for example (trust-to-untrust, untrust-to-trust).

Because of the stateful nature of ZBFW it is not necessary to define rules for
the return traffic of a connection already allowed in the opposite direction.

.. code-block:: none

  zone-pair security <pair-name> source <src-zone> destination <dst-zone>
    service-policy type inspect <policy-name>

.. rubric:: Assign zones to interfaces

The last step is to put the interfaces in the appropriate zone:

.. code-block:: none

  interface <type> <slot/num>
    zone-member security <zone-name>


Verification
------------

To see what interfaces are in a given (or all zones):

.. code-block:: none

  show zone security [<zone-name>]

To see the policy attached to a given zone or zone-pair:

.. code-block:: none

  show zone-pair security [source <src-zone-name>] [destination <dst-zone-name>]

To see the statistics for a specific zone pair, use the following command:

.. code-block::

  show policy-map type inspect zone-pair <zone-pair-name>
