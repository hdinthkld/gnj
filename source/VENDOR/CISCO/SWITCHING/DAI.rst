.. _cisco_dai:

##################################
Cisco Dynamic ARP Inspection (DAI)
##################################

Overview
########

Within a Layer 2 broadcast domain, hosts can be vulnerable to MAC spoofing attacks as there is is no way to verify that indeed the host that replied to a ARP request is indeed to host who owns the required IP address.

Hosts will also accept a gratuitous ARP reply even when they have not requested this information leading to a compromise of the hosts ARP cache.

Cisco's Dynamic ARP Inspection (DAI) feature can help prvent these types of attacks by ensuring only valid ARP requests and response are relayed.  It does this by relying on an existing trusted database, either statically configured or via the DHCP snooping databae.

Hosts are considdered either trusted or untrusted.  Commonly connections between other switches are trusted whereas those connected to host devices are untrusted. Trusted ports bypass the DAI security checks.

DAI should be enabled on all switches as if one switch does not have DAI enabled and is connected to a trusted port, even if the other switch has DAI enabled it will trust all the ARP/MAC information from the non-DAI switch.

DAI also can be used to prevent ARP DoS attacks by rate limiting the number of ARP packets that are incoming. By default the rate is 15 pps on untrusted interaces. If this rate is existed the port is put in an error disabled state and by default manual intervention is required to recover the port but error disable recovery can also be enabled to recover the port after a given period of time.

DAI can be combined with an ARP ACL to filter packets irrelevent of the DHCP snooping database.  If a packet is denied by the ARP ACL is will be dropped irrespectivve of any entries in the DHCP snooping ddatabase.

Implementation
##############

By default DAI is disable on all VLANS and all interfaces are configured as untrusted.

The default rate limiting of incoming ARP packets is 15pps on untrusted interfaces with a burst interval of 1 second.  There is no rate limiting applied on trusted interfaces.

No additional validation checks are performed by default.

DAI only checks packets as they are incoming, nothing is checked for packets leaving an interface.

DHCP Snooping must be enabled prior to enabling DAI. If DHCP Snooping is not enabled or in  non-DHCP environment, use ARP ACLS to permit or deny packets on a per-vlan basis.

Configuration Steps
===================

#. Enable DHCP Snooping (if required)
#. Enable DAI on the VLAN(s)
#. Configure the DAI interface trust state
#. Applying ARP ACLs for DAI Filtering
#. Configure ARP Packet Rate Limiting
#. Enabling DAI error-disabled recovery
#. Configure additional validation
#. Configure DAI Logging

.. rubric:: Enable DHCP Snooping

If DAI needs to be able to check the DHCP database, ensure the DHCP snooping is
enabled on the same VLAN as DAI and also enabled globally on the switch.

Refer to the :ref:`cisco_dhcp_snooping` section for further information

.. rubric:: Enable DAI on the VLAN(s)

DAI is enabled on a per-VLAN basis not globally as follows:

.. code-block:: none

  ip arp inspection vlan {vlan-id> | <vlan-range>}

.. rubric:: Configure the DAI interface trust state

By default all interface are untrusted, for ports connected to other switches the ports should be configured as trusted.

.. code-block:: none

  interface <type/num>
    ip arp inspection [trust | untrust]


.. rubric:: Apply ARP ACL for DAI filtering

If you want to specifically deny/allow certain MAC Addresses irrespective of the DHCP Snooping database an ARP ACL can be used on a per VLAN basis.

.. code-block:: none

  ip arp inspecion filter <arp-acl-name> vlan {vlan-id> | <vlan-range>} [static]

If the static keyword is specified an explicit deny will be used meaning that any entries in the DHCP Snooping database will not be used.  Without this the DHCP snooping database will be consulted to determine if a packet should be denied or not.

.. rubric:: Configure ARP rate limiting

Configuring rate limiting can assist with prevenint ARP based denial of service attacks such as DoS and CAM table floooding.  By default if more than 15 ARP packets are seen per second it will be seen as an attack and the port will be error disabled.

.. code-block:: none

  interface <type/num>
    [no] ip arp inspection limit {rate <pps> [burst interval <seconds>] | none}

Using the *no* version of the command does not disable rate limiting, it simply returns it to the default rate-limiting value.

.. rubric:: Enable DAI Error-Disabled Recovery

To ease the administrative burden of manually recovering ports due to DAI, the switch can be configured to automatically recover the port after a certain amount of time:

.. code-block::
    errdisable recovery cause arp-inspection

.. rubric:: Enable additional validation

By default DAI will discard ARP packets with invalid IP-to-MAC address bindings. Additional verification can be enabled for:

* Destination MAC (Verifies the target MAC in Ethernet to target MAC in ARP)
* Sender/Target IP addresses (Checks for invalid/unexpect IPS, such as wildcard or broadcast)
* Source MAC (verifies source MAC in ethernet against sender MAC in ARP body for both requess and responds)

.. code-block:: none

  ip arp inspection validate {[dst-mac] [ip] [src-mac]}

.. rubric:: DAI Logging

If a packet is dropped by DAI an entry is logged in the buffer which is rate-controlled.  This entry will include the receiving VLAN, port number, source and destination IP an MAC details.

Because of the rate control, a single log entry may represent more than one packet.  If many packets on the same VLAN with same ARP parameters, DAI combines this into a single log buffer entry.

The buffer size can be configured with:

.. code-block:: none

  ip arp inspection log-buffer entries <number>

The aggregation of events can be controlled through:

.. code-block:: none

  ip arp inspection log-buffer logs <num-of-msgs> interval <seconds>

For example, if the number of messages is set to 12 and the interval is set to 2 seconds, 12 messages will be logged every 2 seconds (assuming an event occurs).

The types of log entries can also be controlled as follows:

.. code-block:: none

  ip arp inspection vlan <vlan-range> logging {acl-match {matchlog | none} | dhcp-bindings {all | none | permit}}

Monitoring and Troubleshooting
##############################

The follow commands can be useful for troubleshooting and monitoring DAI:

.. code-block:: none

  ! display the ARP ACLs
  show arp access-list  [<acl-name>]

  ! Displays the current DAI status and ACL configuration per vlan as well as any additional validations
  show ip arp inspection vlan {vlan-id> | <vlan-range>}

  ! Displays the current trust stte, rate and burst interval of each interface
  show ip arp inspection interfaces

  ! Disables the recovery status for the various causes
  show errdisable recovery

  ! Displays the ARP inspection log
  show ip arp inspecion log

Further Reading
###############

The following documentation provides further information:

15.1SY Supervisor Engine 2T Software Configuration Guide [c6]_
