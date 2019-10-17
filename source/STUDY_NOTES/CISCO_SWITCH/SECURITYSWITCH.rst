*******************************
Cisco - Securing Switch Acccess
*******************************

.. _switch_portsecurity:

Port Security
=============

- Controls access to a port based on MAC Address
- MAC addresses are statically configured or dynamically learned
- Limits the number of stations that can be used on a given port, by default 1
- MAC addresses learned as hosts transmit frames
- Learned addresses are lost upon reboot by default unless "Sticky Addresses" is enabled
- By default learned addresses do not age out, this can be configured

Violation Actions
-----------------

- Shutdown (Default)

  * Port put in errdisabled state
  * No traffic flow
  * Manual or error disabled recovery needed

- Restrict

  * Port stays up
  * Frames for invalid MACs are dropped
  * Sends SNMP Trap

- Protect

  * Port stays up
  * All traffic from violated MACs dropped
  * No record of violation recorded


Port-Based Authentication
=========================

- Combination of AAA Authentication and Port Security
- Based on IEEE 802.1x
- User is required to successfully authrnticate before traffic will flow
- Requires support on the switch (Supplicant) and the end user device (client)
- Client configured with 802.1x will still allow traffic to flow when switch is not configured to authenticate
- Uses EAP Over LAN (EaPOL), A layer 2 protocol
- Port returns to an unauthorised state when end user logs out or disconnects port
- Authentication is done from the switch to an authentication server using RADIUS
- Switch ports can be configued into 1 of 3 states

  * Force Authorised - No authentication needed (default)
  * Force Unauthorised - Port will never successfully authenticate
  * Auto - Requires sucessful 802.1x authentication

- By default only a single host is supported on an authenticated port, muliple hosts can be configured

Configuration Steps
-------------------

- Enable AAA Globally On the switchh
- Define RADIUS servers
- Define Authentication Methods for 802.1x
- Enable 802.1x globally on the switch 
- Configure individual ports to use 802.1x authentication by changing them from "Force Authorised" to "Auto"

.. _switch_stormcontrol:

Using Storm Control
===================

- Switches allow networks to be more efficient by only sending frames to the host(s) that need them
- Swiitches cannot optimise the all frame times and must flood them

  * Broadcast
  * Multicast
  * Unknown Unicast

- Storm Control helps by preventing hosts sending excesive amounts of these packet types
- Configured per interface
- Individual thresholds for each frame type
- Thresholds are specified as a percentage of interface bandwidth in either bps or pps
- hosts breaching thresholds can cause certain actions to occur

Storm control actions
---------------------

- Drop (Default) - Exceeding frames are dropped
- Shutdown - Port is placed in error disabled state
- Trap - Frames dropped and SNMP trap sent

Best Practices for securing Switches
====================================

- Configure secure passwords
- Use system banners
- Secure/Disable the web interface
- Secure the console ports
- secure telnet/SSH access
- Use SSH in preference to Telnet where supported
- Use SNMPv3 where possible and restrict to read-only access
- Secure unused switch ports
- Secure STP operation (e.g. BPDU Guard)
- Secure use of CDP/LLDP to only connections with trusted devices

Configuring Port Security
=========================

**Enable Port Security**

::

  interface <name>
    switchport port-security

**Define number of allowed MAC addresses on a port**

::

  interface <name>
    switchport port-security maximum <number>

**Set Dynamically learned addresses to be sticky**

::

  interface <name>
    switchport port-security mac-address sticky

**Manually define the MAC address permitted on a port**

::

  interface <name>
    switchport port-security mac-address <mac>

**Define the action tkane upon detected  an unknown MAC**

*NOTE: Default action is "Shutdown"*

::

  interface <name>
    switchport port-security violation {shutdown|restrict|protect}

**Clear MAC addresses from the port cache**

::

  clear port-security {all | configured | dynamic | sticky } [address <mac> | interface <name>]

**Shows status for an interface**

::

  show port-security [interface <name>]

**Show summary of error disabled interfaces**

::

  show interfaces status err-disabled

**Manually restore an interface**

::

  interface <name>
    shutdown
    no shutdown

**Configure Learned MAC address ageing**

*NOTE: Disabled by default*

::

  interface <name>
    switchport port-security aging {time <minutes> | type {absolute | inactivity}}

Configuring Port-Based Authentication
=====================================

**Enable AAA Globall**

::

  aaa new-model

**Define external RADIUS servers**

::

  radius-server host {<hostname> | <ip>} [key <string>]

**Define authentication method used for 802.1x**

::

  aaa authentication dot1x default group radius

**Enable 802.1x supplicant on the switch**

::

  dot1x system-auth-control

**Set The authentication mode on an interface**

::

  interface <name>
    dot1x port-control {force-authorised|force-unauthorised|auto}

**Allow multiple hosts on a single port**

::

  dot1x host-mode multi-host

**Verify 802.1x operation**

::

  show dot1x all

Configuring Storm Control
=========================

**Enable Storm Control Thresholds**

::

  interface <name>
    storm-control {broadcast|multicast|unicast}
      level {<level> [<level-low> | bps <bps> [<bps-lower>[ | pps <pps> [<pps-lower>]}

**Enable Additional Actions When Thresholds Breached**

*NOTE: Default is to drop exceeding packets*

::

  interface <name>
    storm-control action {shutdown | trap}

**Display Storm Control Status**

::

  show storm-control [<interface>] [broadcast|multicast|unicast]
