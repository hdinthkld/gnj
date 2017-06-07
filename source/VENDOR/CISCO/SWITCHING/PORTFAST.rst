##############
Cisco PortFast
##############

Overview
--------

The Cisco PortFast feature is not directly intended for security but as it is referenced
by a number of the other features, a description of its operation is important.

When Spanning-Tree runs a port has to transition through a number of states before being
able to transmit or receive traffic.  This can cause problems for end-user devices as
whilst this negotiation is running it's possible for the device to assume it's not connected
to a network and make alternative decisions.  Some of the problems that can occur:

* Failure to receive DHCP addresses
* Failure to receive response from Windows Domain Controller leading to local logins
* Failure to detect trusted network, resulting in the device putting itself in a locked
  down state.

Whilst the above problems are not caused by any fault in the network and can often be fixed
by adjusting various timers and so forth in the operating system of the end user device, 
the blame useful falls on the network team and it is up to them to resolve the issue.

Cisco thankfully came up with a solution to this in the name of PortFast.

PortFast works by skipping the Listening and Learning States when a port is plugged in
to a device, immediately transitioning it to the Forwarding state.

This could sound risky as if no learning has taken place, how can a loop be detected. The
good news is that the switch is smart and continues to listen for BPDUs on the PortFast
enabled port.  As soon as a BPDU is seen on the port it immediately looses it's PortFast
status and has to transition through all the spanning-tree port states.

An additional benefit to PortFast is than when enabled, if the port changes state (such
as users unplugging their devices) it does not cause a Topology Change Notification (TCN). 
This helps to avoid constant STP recalcuations and therefore gives a more stable topology. 
In a stable network, the number of topology changes recorded per VLAN should be minimal so
a large increase should be cause for investigation and not end up being just because of 
users going about their normal business.

By default only Access ports can be configured with PortFast but through an additional
option, trunk ports can be made PortFast as well.  This is useful where a non-user device
exists and needs to transition quickly.  In the case of the new Virtualised Infrastructure
it is very useful as often a VM Host will be configured to use multiple VLANs and therefore
needs to be a trunk port.  Care must be taken however that a loop is not caused on the
virtualisation switching layer as many of those virtual switches do not fully implement
the Spanning Tree Protocol.

Implementation
---------------

PortFast can be enabled globally on all access ports using:

.. code-block:: none

  spanning-tree portfast default

It can also be enabled on a per-port basis as follows:

.. code-block:: none

  interface <type> <slot/num>
    spanning-tree portfast [trunk]


Verification
------------

show interface switchport

Troubleshooting
---------------

Because a port automatically transitions into a non-portfast state when BPDUs are detected,
no action should usually be detected.

The most likely issue is where PortFast is not enabled and users are complaining about a
lack of network access (such as failure to receive a DHCP address due to it timing out).

