.. _cisco_portsecurity:

###################
Cisco Port Security
###################

Overview
--------

In some of the previous sections its been discussed how possibly dumb or maliscious/rogue
devices can cause problems in the network, many of the earlier techniques have used a
means of detecting that those devices are (or at) there such as the prescense or lack of
BPDUs. How then can a device be detected that is not actively probing or sending out
other data.

One of the earliest solutions to this is Cisco Port Security.

Port Security works by monitoring and recording the MAC addresses seen on a configured port.
It also tracks how many MACs have been seen on the port. This can be used in a number of
ways:

* Preventing more than a configured number of MAC addresses from being allowed on the port
* Disabling a port if a known secure MAC address is seen on another port.
* Dynamically learning the MAC addresses seen on the port and not allowing any further
  MAC addresses to be learnt until the others have timed out.

MAC Addresses can either by statically configured or learnt dynamically. Even if the port
where a dynamically learnt MAC address exists goes down, it will still remain in the
configuration and will be saved if the running configuration is wrote to the startup
configuration.

A number of actions can be taken when a situation arises

* Drop Packets with unknown source addresses until sufficient number of secure
  MAC addresses has dropped below the maximum value (Protect)
* Drop Packets with unknown source addresses and also increase the Security Violation
  Counter (Restrict)
* Put the interface in the error-disabled state and send an SNMP trap notification (Shutdown)

By default a port is configured with a maximum of one (1) MAC address and the port violation
mode is set to Shutdown.  Port Security however is not enabled by default.

Implementation
--------------

Port Security can be configured only on Layer 2 Ports, not those running routed mode.  It is
however supported on both Access and Trunk ports.

It must be enabled per-port.  For an Access Port:

.. code-block:: none

  interface <type> <slot/num>
    switchport
    switchport mode access
    switchport port-security
    switchport port-security violation {protect | restrict | shutdown}
    switchport port-security maximum <no-of-mac-addresses>

    ! Configure Sticky learnt MAC addresses
    switchport port-security mac-address sticky

    ! Manually configure a secure MAC address
    switchport port-security mac-address <mac-address>

    ! Allow the secure MAC addresses to timeout
    switchport port-security aging type { absolute | inactivity }
    switchport port-security aging time <minutes>

Verification
------------

To verify the Port Security use:

.. code-block:: none

  show port-security [interface {vlan <vlan-id> | <type> <slot/num>} [address]


Troubleshooting
---------------

It is not possible to have the same secure MAC address configured on two different switch
ports. This includes when multiple switches are combined into a switch stack.

In a high availability setup (such as with Load Balancing or EtherChannels) do not configure
Port Security on those ports, if you do at least ensure that the MAC address aging is
enabled with a very low value.  The minimum value that can be set for this is 1 minute so an
outage should still be expected unless multiple servers are used with a method not involving
shared MAC Addresses.
