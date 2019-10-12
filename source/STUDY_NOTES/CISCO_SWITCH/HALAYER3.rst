*********************************
Cisco - Layer 3 High Availability
*********************************

Overview
========

- Switches have support for router redundancy protocols
- Sometimes called First Hop Redudancy Protocols (FHRP)
- Protocols Supported

  * Hot Standby Router Protocol (HSRP)
  * Virtual Route Redundancy Protocol (VRRP)
  * Gateway Load Balancing Protocol (GLBP)

- Used to avoid a single router becoming a single point of failure for traffic needing to leave the subnet/VLAN

Packet Forwarding Review
========================

- Hosts use ARP to find devices on the local subnet
- An Intermediate System (a router) is required to reach another subnet
- Hosts that understand routing will ARP for the gateway MAC to send packets to 
- Hosts that don't understand routing will ARP for all IPs (even remote ones) the route must reply
  with it's own MAC in the form of a proxy ARP
- The gateway/IS/Router availability is critical for the network to function

Hot Standby Router Protocol (HSRP)
==================================

- Cisco Proprietary
- Documented in RFC 2281
- One router is elected as "Active", another as "Standby"
- Routers other than the Active/Standby remain in a LISTEN state
- Hello messages exchanged so other routers know of their existance (default: 3 second interval)
- Routers assumed down if they miss 3 hello interfaces (default: 10 seconds)
- Uses  multicast 224.0.0.2 ("All Routers") over UDP port 1985
- Routers arranges into groups (ID 0 - 255)
- Upto 16 groups supported on the switch as a whole but be reused on multiple VLANs

Router Election
---------------

- Priority Value (0-255), Default: 100
- Highest priority becomes active router, highest IP used as a tie break

HSRP State Transition
---------------------

- HSTP States
  * Disabled (not a true HSRP state, used for interfaces that are adminitrative down)
  * Learn
  * Listen
  * Speak
  * Standby
  * Active

- Only router in "Standby" state monitors hellos from the "Active" router 
- Election of new standby occurs after current standby assumes active role
- By default the previous active router cannot be come active until current active fails
- Preemption can be used to ensure the highest priority router is always active
- Hosts are configured to talk to a virtual IP, not an IP assigned directly to a single router
- Virtual IP used has a MAC address of 0000.0C07.ACxx (xx = Group ID)

Authentication
--------------

- Used to avoid peers with default configuration or unauthorised devvices participating in the HSRP group

- Supported Autentication Methods

  * Plain Text
  * MD5

- Plain Text Authentication

  * Offers basic protection to prevent misconfigured peers participation in the group
  * Default key string is cisco

- MD5 Authentication

  * Authentication hash computered on a part of each HSRP message
  * Secret key known own to legitimate HSRP group peers
  * Only hash is sent across in packets
  * Hash used to validate message contents
  * Key string (up to 64 characters) must be configured on each HSRP router in the group

Conceding The Election
----------------------

- Used to sway the election in the event of interface and (e.g. route peering) failures
- Gateway will reduce it's priority, making it less likely to be the active router

Load Balancing with HSRP
------------------------

- Multiple HSRP groups can exist on a subnet/vlan, each with a unique ID
- Both routers on the subnet can be used at the same time whilst still providing redundancy
- Each router is configured as the primary for it's own group and secondary for the peer routers group
- Hosts must be configured to use the most approrpiate gateway either manually or via DHCP


Virtual Router Redundancy Protocol (VRRP)
=========================================

- Standards based protocol, documented in RFC 2338
- One router is appointed as "Master" router, others are "Backup" routers
- Master based on highest priority value (1-254), default: 100
- Uses virtual IP and MAC with prefix 0000.5E00.01xx (xx = Group ID)
- Group ID is 0 to 255
- Advertisements sent at 1 second intervals, interval can be learned from master router
- Premption enabled by default
- Advertisements sent with multicast IP 224.0.0.18 using IP protocol 112
- Introduced in IOS 12.0(18)ST, not supported on all switch platforms
- Can use inteface tracking 
- Multiple groups supported per VLAN for load balancing


Gateway Load Balancing Protocol (GLBP)
======================================

- HSRP/VRRP can provide load balancing but requires external assistance to point hosts as
  the appropriae virtual IP
- GLBP provides both redudancy and load balancing without needing client or server configuration
- Cisco Proprietary
- Introduced in IOS 12.2(14)5, no consistent support on switch platforms
- Switches/Routers assigned to a common group
- All routers participate in forwarding a portion of the traffic
- Load balancing achieved through virtual MAC addresses
- Each client will receive a different ARP reply even though same gateway IP is used

Active Virtual Gateway (AVG)
-----------------------------

- Only one router is elected as the AVG
- Election based on highest priority, then highest IP
- Responsible for answering all ARP requests
- MAC returned depends on configured load balancing method
- Response for assigning  MAC to router in the group (AVF)
- Upto 4 virtual MAC addresses supported
- AVG also assigned secondary roles
- Group ID can be 0 - 1023
- Priority can be 1 - 255, default: 100
- Premption is supported, not enabled by default
- Hellos sent every 3 seconds by default
- Peer assumed failed after holdtime expires (default: 10 seconds)
- Holdtime should be 3 times the hello interval
- Timers only need to be configured on the AVG which will advertise to other routers

Active Virtual Forwarder (AVF)
------------------------------

- Responsible for forwarding traffic received from clients
- Virtual MAC prefix of 0007.B4xx.xxyy
  * xx.xx = 0 bits followed by 10 bit group ID
  * yy = 8-bit virtual forwarder number

- Handling AVF Failure

  * If hellos are missed, AVG will assign AVF role to another router
  * AVG will continue to process traffic on olf MAC until "Redirect Timer" expires
  * Redirect timer by default is 600 seconds
  * When timeout expires old MAC and AVF are flush from all GLBP peers
  * Clients must refresh ARP to find new MAC address after it has been flushed

- Weighting

  * Used to determine which router becomes the AVF for a virtual MAC
  * Weight value between 1 and 254, default: 100
  * weight decreased as interfaces go down
  * AVF role is given up if weight is below lower threshold
  * Router can resume AVF role when weight is above upper threshold
  * GLBP must be configured with interfaces to track
  * AVF cannot preempt another AVF with a higher weight

GLBP Load Balancing
-------------------

- MAC address handed to clients in a deterministic fashion
- Supported load balancing methods
  
  * Round Robin (Default) - Even traffic load across all AVFs
  * Weighted - AVFs receive traffic based on configured weight values
  * Host Dependant - host is given consistent MAC every time


HSRP configuration
==================

**Specify Router Priority**

*NOTE: Default priority is 100*

::

  interface <name>
    standby <group> priority <value>

**Set HSRP Timers**

*NOTE Default timers are 3 seconds (hello) and 10 seconds (holdtime)*

::

  interface <name>
    standby <group> timers [msec] <hello-interval> [msec] <holdtime>

**Enable higher priorty router to take over from current active router**

::

  interface <name>
    standby <group> prempt [delay [minimum <seconds] [reload <seconds>]]

**Configure plain-text authentcation**

::

  interface <name>
    standby <group> authentication <string>

**Configure MD5 authentication**

::

  key chain <keychain-name>
    key <number>
      key-string [0|7] <string>

  interface <name>
    standby <group> authentication md5 key-chain <keychain-name>

**Configure Priority changed based on interface status**

*NOTE: Default decrement value is 10*

::

  interface <name>
    standby <group> track <interface-name> [<value-to-decrement>]

**Specify Virtual IP To Use For Group**

::

  interface <name>
    standby <group> ip <ip> [secondary]

**Enable HSRP for IPv6**

::

  interface <name>
    standby version 2
    standby ipv6 autoconfig

**Verify HSRP Status**

::

  show standby [brief] [vlan <id> | <interface-name>]

VRRP configuration
==================

**Set Router Priority**

::

  interface <name>
    vrrp <group> priority <level>

**Set advertisement interval**

::

  interface <name>
    vrrp <group> timers advertise [msec] <interval>

**Configure advertisement learning**

::

  interface <name>
    vrrp <group> timers learn

**Disable/Enable Prempting**

*NOTE: Enabled by default*

::

  interface <name>
    vrrp <group> preempt [delay <seconds>]

**Set Authentication String**

::

  interface <name>
    vrrp <group> authentication <string>

**Assign Virtual IP**

::

  interface <name>
    vrrp <group> ip <ip> [secondary]

**Enble Interface Tracking**

::

  interface <name>
    vrrp <group> track <interface-name> [decrement <value>]

**Check VRRP Status**

::

  show vrrp [brief] [all]

GLBP Configuration
==================

**Assign Priority To A Router**

::

  interface <name>
    glbp <group> priority <level>

**Enable Prempting**

*NOTE: Disabled by default*

::

  interface <name>
    glbp <group> preempt [delay minimum <seconds>]


**Set Timers**

*Default: 3 second hello, 10 second holdtime*

::

  interface <name>
    glbp <group> timers [msec] <hello> [msec] <hold-time>

**Set AVF Redirect/Timeout Timers

::

  interface <name>
    glbp <group> timers redirect <redirect-interval> <timeout>

**Configure Tracking Object**

::

  track <id> interface <name> {line-protocol | ip routing}

**Set Weighting Thresholds**

*Note: Default max 100*

::

  interface <name>
    glbp <id> weighting <max> [lower <lower-weight>] [upper <upper-weight>]


**Define tracking criteria**

::

  interface <name>
    glbp <group> weighting track <id> [decrement <value>]

**Set Load Balancing Method**

::

  interface <name>
    glbp <group> load-balancing [round-robin|weighted|host-dependent]

**Set Virtual IP**

*NOTE: Must be configured on the AVG, learnt by other routers*


::

  interface <name>
    glbp <group> ip [<ip> [secondary]]

**Enable GLBP for IPv6**

::

  interface <name>
    glbp <group> ipv6 autoconfigure

**Verify GLBP**

::

  show glbp [<group>] [brief]