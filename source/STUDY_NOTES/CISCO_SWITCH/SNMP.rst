*****************************************
Cisco - Monitoring Campus Networks - SNMP
*****************************************

SNMP Overview
=============

- Simple Network Management Protocol (SNMP)
- Data is stored in a Management Information Base (MIB) Database
- Provides a means to monitor devices and obtain various statistics
- The manager will communicate with the device (agent) over UDP Port 161 (Manager to Agent)
- Agent will send messages to the manager over UDP Port 162 (Traps and Informs)

SNMP Versions
-------------

- Valid versions are 1, 2c and 3
- SNMP version 1 (RFC 1157)

  * Simple Get & Set requests
  * Supports Traps
  * Uses simple "community string" to gain access to agent
  * No encryption, community string sent in plain text

- SNMP version 2c (RFC 1901)

  * Adds 64-bit counters
  * Bulk requests supported
  * Added acknowledged Traps, called Informs

- SNMP version 3 (RFC 3410 - 3415)

  * Authentication through usernames
  * Users can be organised into groups
  * Users/Groups can have restricted access through "Views"
  * Encryption supported
  * Data Integrity guarantees

SNMP Components
---------------

- Manager

  * Polls And Receives data via SNMP
  * Runs in a central location

- Agent

  * Runs on the network device
  * Respoonses to SNMP Polls
  * Sends "unsolicited" alerts as either Traps or Informs


SNMP Request Types
------------------

- GET Request - Poll for one specific MIB value using the OID
- GET NEXT Request - Poll for the next value following an initial get request
- GET BULK Request - Poll for entire table or list of values
- SET Request - Asks the device to set a MIB variable to a specific value


SNMP Traps And Informs
----------------------

- The device sends real time alerts to the manager
- Use UDP port 162
- A Trap is a one-way acknowledgement that something significant has happened.  No acknowledgement is expected
- An Inform operates same as a Trap but requires an an acknowledgement to be received from the manager by echoing it back

Configuring SNMP version 1/2c
=============================

**Define Access to Agent by IP**

::

  access-list <acl-number> permit <ip>

**Specify Who can Access the Switch via SNMP**

::

  snmp-server community <string> [ro | rw] [<acl-number>]

**Define where to send trap/informs**

::

  snmp-server host <ip> <community> [<trap-type>]
  snmp-server host <ip> [inform] version 2c <community>

Configure SNMP version 3
========================

*NOTE: Use an access-list list restrict who can acess the SNMP agent via IP*

**Define a View for which MIB vales can be read/write**

::
  
  snmp-server view <name> <oid-tree>

**Create A User Group**

::

  snmp-server group <group-name> v3 {noauth | auth | priv} [read <view>] [write <view>]
                    [notify <view>] [<acl-number>]

**Define User and Map to Group**

::

  snmp-server user <user-name> <group-name> v3 auth {md5|sha} <auth-password>
              priv {des|3des|aes{128|192|256}} <priv-password> [<acl-number>]

**Define where to send trap/informs**

::

  snmp-server host <ip> [informs] version 3 {noauth|auth|priv} <username> [<trap-type>]

