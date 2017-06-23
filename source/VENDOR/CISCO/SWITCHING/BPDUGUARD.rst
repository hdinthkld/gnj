.. _cisco_bpduguard:

###########################
Cisco BPDU Guard and Filter
###########################

Overview
--------

It a badly managed network a device capable of sending BPDUs can be plugged in pretty much
anywhere.  This can cause issues for the network administrator trying to maintain a stable
environment for there users, especially when it could be the users causing problems for
themselves.

It is not uncommon for users to need more than the single wall port that the IT team has
provisioned for them.  This kind of problem is even more prevelant in the engineering and
scientific work place where users often need to have network access for more than one device.

The impatient user or helpful IT desktop support team member often solves this problem by
plugging in a cheap (most likely unmanaged) hub or switch into the port.  The users problem
is now solved as he can plug in any many devices as the "rogue" switch has available.

The problem with this is that if the switches in the network (including those managed by
the IT team) are deployed with default configurations, they most likely will have a default
Bridge Priority this could be the same as the rogue switch and results in sub-optimal
network topologies where the root bridge is seen through a low bandwidth port or device.

Another issue is that some cheap hubs (and switches) don't correctly implement the spanning
tree protocol and can cause loops in the network where more than one cable from the rogue
device is connected to the managed network via two seperate cables.  In practice either
the managed switch or the rogue switch should disable one of its ports to prevent the
loop but this doesn't always happen (and hubs don't even know how).

BPDU Guard
==========


Cisco has created an enhacement to deal with the first issue known as BPDU Guard.  A second
feature know as Loop Guard which helps deal with the second issue will be discussed
seperately.

Unlike with Root Guard, the BPDU Guard features don't given any consideration to the
values in the BPDU. If any BPDU is seen on a port with BPDU Guard enabled, it is immediately
put into the *errdisable* state. When this occurs a syslog message is logged to alert
the network team:

*%SPANTREE-2-RX_PORTFAST:Received BPDU on PortFast enable port. Disabling <type> <slot/num>*

Recovery is not automatic but a timeout interval can be configured to return the port to
an operational state after a given amount of time.  If the port is still sending BPDUs it
will simply be blocked again for another interval.


BPDU Filter
===========

A companion or alternative feature to BPDU Guard is to use BPDU Filter.

Unlike BPDU Guard, the BPDU Filter does not shut the port down instead it simply prevents
the port from sending or receiving BPDUs. If a BPDU is received on PortFast enabled
interface the interfaces loses it's PortFast status and BPDU filtering is disabled.

It should be noted that enabling this feature effectively disables spanning-tree on the port
and therefore could result in loops in the network topology.

BPDU Filter can be useful in preventing BPDUs from going beyond the managed network. Some
known attacks exploit receiving BPDUs and deliberately setting their Priority lower than
the root bridge.

In general it is recommended to deploy BPDU Guard and not BPDU Filter as it leads to a more
controlled and predicatable network topology.


Implementation
---------------

BPDU Guard is deployed alongside PortFast. This is useful as it is assumed that any
port which has PortFast enabled should generally be an end user device, not a switch so
if a user were to plug a switch into the port it is correct that the port should be
disabled and the network team notified so they can take corrective action.

To enable BPDU Guard/Filter on all ports that have PortFast enabled, use the following
command:

.. code-block:: none


  ! Use or or the other, not both
  spanning-tree portfast bpduguard default
  spanning-tree portfast bpdufilter default

At the port level BPDU Guard/Filter can be enabled individually:

.. code-block:: none

  interface <type> <slot/num>
    spanning-tree portfast

    ! Use one not both of these options
    spanning-tree bpduguard enable
    spanning-tree bpdufilter enable

If it is preferred to have the port return to an operational state without the network
teams intervention, a recovery interval can be configured with:

.. code-block:: none

  errdisable recovery cause bpduguard
  errdisable recovery interval <seconds>

If the interval is not specified it defaults to 300 seconds (5 minutes).

Verification
------------

To confirm if BPDU Guard has been enabled, use this command:

.. code-block:: none

  show spanning-tree summary totals
