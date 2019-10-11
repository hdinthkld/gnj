***************************************************************
Cisco - Monitoring Campus Networks - IP Service Level Agreement
***************************************************************

IP SLA  Overview
=================

- IP Service Level Agreement (IP SLA)
- Used to gather  information on how the work is performing from a point actually within the network
- Takes the place of remote probes, all managed from central location

- Legacy names

  * Response Time Reporter (RTR)
  * Service Assurance Agent (SAA)

- Basic tests only require the source device to be configured
- Advanced tests (E.g. Jitter) need a peer device to be configured as a "Responder" in order to collec the statistics

- Control connection is setup between source and responder to carry out

  * Test setup
  * Time stamping of packets to account for latency

- NTP is recommended to keep clocks synchronised
- Tests can be triggered via SNMP as long as write access is permitted

Features
--------

- Generate SNMP Trap when thresholds are exceeded
- Schedule further IP SLA tests when thresholds are exceeded
- Track IP SLA to trigger NHRP failover
- Gather voice quality measurements frm all over the network

Supported Tests
---------------

- ICMP-Echo
- Path-Echo
- Path-Jitter (requires responder)
- DNS
- DHCP
- FTP
- HTTP
- UDP-Echo
- UDP-Jitter (requires responder)
- TCP-Connect

Configuring IP SLA
==================

*NOTE: In IOS prior to 12.2(33) the commands has the "ip sla monitor" prefix*

**Enable responder on destination**

::

  ip sla responder

**Enable IP SLA Authentication**

::

  key chain <name>
    key-string <string>

  ip sla key-chain <name>

**Create a new test**

*Note: In order to modify a test the existing test must be unscheduled then deleted before being recreated*

::

  ip sla <number>
    <test-type> <parameters>
    frequency <seconds>

**Schedule the test to run**

::

  ip sla schedule <number> [life {forever | <seconds>}
         [start-time {<hh>:<mm>:<ss>] [<month> <day> | <day> <month>]
         | pending | now | after <hh>:<mm>:<ss>}]
         [ageout <seconds>] [recurring]


Configure Specific Test Types
=============================

**Setup An ICMP Echo Test**

::

  ip sla <number>
    icmp-echo <ip> [<src-ip>]
    frequency <seconds>

**Setup UDP Jitter Test**

*NOTE: Ensure that IP SPA Responder is running on the remote device*
::

  ip sla <number>
    udp-jitter <dst-ip> <dst-udp-port> [source-ip <source-ip>]
               [source-port <src-port>] [num-packets <packets>]
               [interval <ms>]

  ip sla <number>
    udp-jitter <dst-ip> <dst-udp-port> codec
               {g711alaw|g711ulaw|g729a}

Advanced IP SLA Usage
=====================

**Enable HSRP Failover based on IP SLA Result**

::

  track <number> ip sla <sla-number> {state | reachability}

  interface <name>
    standby <group> track <sla-number> decrement <value>

Verify IP SLA
=============

**Display Configuration**

::

  show ip sla configuration [<id>]

**Display IP SLA Test Result**

::

  show ip sla statistics [aggregated] [<number>]
