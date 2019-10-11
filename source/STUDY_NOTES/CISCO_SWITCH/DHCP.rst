************************
Cisco - Configuring DHCP
************************

Using DHCP With A Multilayer Switch
===================================

- Dynamic Host Configuration Protocol (DHCP) is defined in RFC 2131
- Provides features to configure hosts with specific settings
- Client/Server model
- Hosts run a DHCP client
- The server will issue a leased IP address to the client that is only valid for a specified time
- Client must renew the IP address before it expires in order to continue using it
- DHCP only works within a single broadcast domain
- Using a DHCP Relay, the request can be forwarded to a DHCP server on a different subnet


DHCP Negotiation Process
========================

- DHCP Discover message sent from client as a Broadcast
- DHCP Offer messsage sent from server as a Broadcast with lease information

  * IP Address
  * Subnet Mask
  * Default Gateway
  * Other options if configured
  * Server IP to identify where offer came from

- DHCP Request from client sent as a broadcast
- DHCP Ack (Acknowledgement) from server, defaults to unicast but can also be broadcast

DHCP Configuration Steps
========================
- Configure Layer 3 address on an SVI
- Configure IP addresses to exclude

  * The switches own IP address and broadcast are automatically excluded

- Configure DHCP Pool
  
  * Must correlate with a configure IP Interface
  * Default lease time is 1 day

Configuring a manual DHCP binding
=================================

- Requires one pool per binding
- The client can request an IP using either it's MAC address or "Client Identifier"
- Client Identifier Format

  * EtherType followed by MAC Address
  * Client ID uses "01" for the Ethernet EtherType

DHCP Options
============

- Allow additional configuration items to be supplied to client
- Each option is identified by a number, followed by a value
- Common DHCP Options

  * 43 - Location of WLC (Value is an IP address) 
  * 69 - Location of SMTP Server (Value is an IP address)
  * 70 - Location of POP3 Mail Server (Value is an IP address)
  * 150 - Location of TFTP server for Cisco IP Phones (Value is an IP address)

- Many options are industry standards but vendors can implement custom options for their needs
- Common options (such as default gateway) have predefined configuration commands so it's not necessary to define the options manually


Configuring A DHCP Relay
========================

- Facilitates locating a DHCP server in a central location where it can be used or multiple subnets/VLANs
- Relay will forward any DHCP broadcasts to the configured servers as a unicast message
- Configured on each Layer 3 interface which faces the client devices
- Most then one DHCP server can be configured for redudancy

Configuring DHCP To Support IPv6
================================

IPv6 Refresher
--------------

- IPv6 can auto-configure the host by discovering a local IPv6 router
- Host learns network prefix to generate a link-local address
- Link local addresses start with FE80::/10
- Link local addresses is made up of prefix  and host MAC address
- Duplicate address detection (DAD) is used to ensure no conflict

Stateless Auto-Configuration
----------------------------

- Clients generate it's own IP address
- 64-bit Layer 3 subnet prefix
- EUI-64 ID (64-bits)

  * Upper half of interface MAC (24 bits)
  * Static Value FFFE (16 bits)
  * Lower half of interface MAC (24 bits)

- Client can learn default router and MTU from the router
- Router advertises it's prescense periodicall or client can request on demand

DHCPv6
------

- Routers specify if DHCPv6 is offered in their advertisements
- Clients can also request the service
- Excluding addresses is not permitting in DHCPv6
- Manual address bindings are not supported

- DHCPv6 Lite

  * Combines stateless auto-configuration wwith options provided by DHCPv6
  * Prefix  not specified in pool to prevent clients obtaining an IP address

- DHCPv6 Relay Agent

  * Permits the DHCPv6 server to be located on a different subnet to the clients

DHCP Server Configuration
=========================

**Exclude addresses from allocation**

::

  ip dhcp excluded-address <start-ip> <end-ip>

**Configure DHCP Pool**

::

  ip dhcp pool <name>
    network <subnet> <mask>
    default-router <ip> [<ip-address> ... ]
    lease { infinite | {<days> [<hours> [<minutes>]]}
    option <number> <value>

**Clear currently active leases**

::

  clear ip dhcp binding {* | <ip>}

**Configure Manual Binding**

::

  ip dhcp pool <name>
    host <ip> <mask>
    client-identifier <client-id>


**Configure DHCP Relay**

*Note: Applied to either a Layer 3 or VLAN interface*

::

  interface <name>
    ip helper-address <server-ip> [<server-ip> ...]


DHCPv6 Configuration
====================

**Configure IPv6 Interface**

::

  interface <name>
    ipv6 address <prefix>/<mask>
    no shutdown

**Configure DHCPv6 Pool**

*Note: To configure DHCPv6 Lite Pool, do not specify "address prefix"
::

  ipv6 dhcp pool <name>
    address prefix <prefix>/<mask>
    dns-server <ipv6-ip>
    domain-name <string>

**Associate DHCPv6 Pool With Interface**

::

  interface <name>
    ipv6 dhcp server <pool-name>

**Configure DHCPv6 Lite On an Interface**

::

  interface <name>
    ipv6 dhcp server <pool-name>
    ipv6 nd other-config-flag

**Specify DHCPv6 Relay**

::

  interface <name>
    ipv6 dhcp relay destination <ipv6-address> [<ipv6-address> ...]


**Clear Active DHCPv6 Bindings**

::

  clear ipv6 dhcp bindings {* | <ipv6-address>}

Verification and Troubleshooting
================================

**Display Current DHCP server Address Assigments**

::

  show ip dhcp bindings

**Debug DHCP transactions from the switch**

::

  debug ip dhcp server

**Display DHCPv6 Address Bindings**

::

  show ipv6 dhcp pool