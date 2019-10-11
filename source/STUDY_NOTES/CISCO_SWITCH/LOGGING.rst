********************************************
Cisco - Monitoring Campus Networks - Logging
********************************************

Logging Overview
================

- Used to monitor events taking place on the switch
- Having accurate time is important to correlate events across the entire network
- Detect failures and assist with troubleshooting

Syslog Messages
===============

- Audit trail describing important events that have occured
- Identify what, when and how an event occured
- Message Format

  * Timestamp
  * Facility Code
  * Severity (0-7, 0 being most severe)
  * Mnumonic
  * Message Text

- Severity Levels (Highest to Lowest Severity)
  
  * Emergencies (0)
  * Alerts (1)
  * Critical (2)
  * Errors (3)
  * Warnings (4)
  * Notification (5)
  * Informational (6)
  * Debugging (7)

Logging Destinations
====================

- Console

  * Defaults to the debugging (7) severity level
  * Only by default logs to serial console

- Internal Buffer

  * Stored in internal memory
  * Lost if switch crashes/powered off/reloaded
  * Disabled by default
  * Default buffer size of 4096 bytes

- Remote Syslog Server

  * Uses UDP over port 51 by default
  * Severity and server IPs must be defined

Adding Timestamps to Syslog Messages
====================================

- Important for viewing non-real time historical events
- Default timestamp is the "Uptime" of the switch
- The Uptime will become more coarse over time (E.g. 3w2d)
- Clock Sync options
  
  * Manually
  * NTP / Authenticated NTP
  * SNTP

- Timezone can be defined as can offset from UTC
- Daylight Saving Time (DST) must be configued Manually

Using NTP to Synchronising with External Time Source
====================================================

- Ensures consisten time across multiple devices
- Accounts for delay during NTP synchronisation
- A hierarchy of servers  can be defined by specifying the "Stratum" value
- Higher Stratums are considered more accurate
- Multiple tiers of NTP servers allow for greater scaleability
- The server configured with the lowest stratum value is preferred over others

NTP Modes
---------

- Server

  * Synchronise with a lowest stratum source
  * Provides time sync to other servers/devices

- Client

  * Syncs its clock with an NTP server

- Peer

  * Exchanges time with another peer device

- Broadcast/Multicast

  * Operates as an NTP server
  * Pushes time information to listening devices
  * Not as accurate as other modes

Securing NTP
------------

- Methods

  * NTP Authentication
  * Restrict access by IP and Activity

- NTP Authentication

  * Does not encrypt data
  * Ensures client is talkin to a "trusted" server
  * Does not restrict access even when "key" is configured

- Restrict access by IP and Activity

  * Configuring an authentication key only validates the server
  * Access List can be used to define what action the listed IP/subnets can carry out

  * Valid Activities

    * Serve-only - Only Sync Request permitted
    * Serve - Sync and control requests, cannot sync
    * Peer - Sync and control requests, can sync time
    * Query-Only - Permit only control queries

- Using SNTP to Synchronise Time

  * A reduced set of NTP functions
  * Operates as client only
  * Time Sync is simplified but less accurate

Configure Logging
=================

**Set Severity For Console Logging** (Default: Debugging)

*NOTE: Severity can be either a number (0 being most severe) or the descriptive name (E.g. critical)*

::

  logging console <severity>

**Set Severity For Internal Buffer** (Default: Disabled)

*NOTE: Severity can be either a number (0 being most severe) or the descriptive name (E.g. critical)*

::

  logging buffered <severity>

**Set Size of Internal Logging Buffer** (Default: 4096 bytes)

::

   logging buffered <bytes>

**Set Severity To Send To Remote Syslog Server**

*NOTE: Severity can be either a number (0 being most severe) or the descriptive name (E.g. critical)*

::

  logging trap <severity>

**Set Syslog Servers To Receive Messages**

*Note: Multiple servers can be configured and all will receive the messages*

::

  logging host <ip>

**Disable/Enable Logging Of Interface Status Changes**

::

  [no] logging event link-status

**Set The Timestamp To Include On Log Messages**

*NOTE: Applies to all logged message irrelevent of logging destination*

::

  service timestamps log datetime [localtime] [show-time-zone] [msec] [year]

**Show Messages In The Internal Bufffer And Logging Settings**

::

  show logging

Configuring Clock On A Switch
=============================

**Manually Set the Client**

*NOTE: Completed from privileged exec mode, not configure mode*

::

  clock set [<hh>:<mm>:<ss>] [<month>] [<day>] [<year>]

**Define The Local Timezone**

::

  clock timezone <name> <offset-hours> [<offset-minutes>]

**Define Daylight Saving Times**

*NOTE: Use one of the below methods**

::

  clock summer-time <name> date <month> <day> <year> <hh>:<mm>
                                <month> <day> <year> <hh>:<mm> [<offfset-hour>:<offset-mins>]


  clock summer-time <name> recurring [<start-week> <day> <month> <hh>:<mm>
                                      <end-week> <day> <month> <hh>:<mm>]
                                     [<offset-mins>]

**Define NTP Server**

::

  ntp server <ip> [prefer] version {3 | 4}]

**Setup NTP Authentication**

::

  ntp authentication-key <number> md5 <string>
  ntp authenticate
  ntp trusted-key <number>
  ntp server <ip> key <number>


**Restrict Access To NTP**

::

  access-list <acl-number> permit <ip> <mask>
  ntp access-group {serve-only|serve|peer|query-only} <acl-number>

**Configure SNTP With Authentication**

::

  sntp authentication-key <number> md5 <string>
  sntp authenticate
  sntp trusted-key <number>
  sntp server <ip> key <number>



**Verifying NTP Synchronisation**

::

  show ntp status

**Display A Summary Of Configured NTP Relationships**

::

  show ntp associations