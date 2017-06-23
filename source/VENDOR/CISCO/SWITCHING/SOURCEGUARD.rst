.. _cisco_sourceguard:

##################
Cisco Source Guard
##################

Overview
########

IP Source Guard (IPSG) Sometimes also known as IP Source Verification, this feature works alongside the DHCP snooping database to prevent a host being able to spoof the IP address of another host on the same network.

When initially enabled o a port, the switch will block all IP traffic received on the interface except for DHCP packets allowed by DHCP snooping. The source IP
lookup table is then used to bind IP addresses to ports. IP taffic with a source IP address in the binding table is allowed, everything else is denied.

It is also possible to configure manual bindings if DHCP is not used.

IPSG is supported on on Layer 2 ports, including access and trunk ports. It can also be configured with sourcce IP address filtering or with source IP and MAC address filtering.

One point to note, whilst IPSG and DAI may appear similar they are actually performing different functions. DAI is intended to prevent ARP spoofing, where as IPSG is intended to prevent IP spoofing.

Implementation
##############

If IP Source Guard is being deployed in a DHCP environment it is necessary to have DHCP snooping enabled first, refer to the :ref:`cisco_dhcpsnooping` section for further details.

Configuration Steps
===================

#. Configure Static bindings
#. Enable IPSG per interface
#. Configure IP Source Guard for Static Hosts

.. rubric:: Configure Static Bindings

For any hosts not using DHCP, it's necessary to configure a static entry.

.. code-block:: none

  ip source binding <mac> vlan <vlan-id> <ip> interface <type/slot>

.. rubric:: Enable IPSG per interfae

.. code-block:: none

  interface <type/num>
    ip verify source [mac-check]

The optional *mac-check* argument causes IPSG to also implement MAC address filering.


.. rubric::  Configure IP Source Guard for Static Hosts

.. code-block:: none

  ip device tracking

  interface <type/num>
    switcport mode access
    switchport access vlan <vlan-id>
    ip verify source [tracking] [mac-check]
    ip device tracking maximum <number>


Monitoring and Troubleshooting
##############################

.. code-block:: none

  show ip verify source

  show ip device tracking {all | interface <type/num> | ip <ip> | mac <mac>}

Further Reading
###############

Security Configuration Guide, Cisco IOS XE Release 3SE (Catalyst 3650 Switches) [c9]_
