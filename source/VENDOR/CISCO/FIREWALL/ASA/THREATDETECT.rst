.. _cisco_asa_threatdetection:

==========================
Cisco ASA Threat Detection
==========================

Overview
========

Available in ASA versions 8.0 and above, Threat Detection provides firewall
administrators with the necessary tools to identify, understand, and stop
attacks before they reach the internal network infrastructure.

It acts like a simple IDS by detecting unusual traffic patterns and
possibily preventing the anomaly traffic from reaching the internal network.

Primarily this features is intended to provide protection to internal hosts
against DoS and DDoS attacks as it can only act on the rates of various
traffic patterns, not on details included in the traffic as would be done
with a signature-based IDS/IPS.

Components
----------

* Basic Threat Detection

* Advanced Threat Detection

* Scanning Threat Detection

Limitations
-----------

* Supported on ASA 8.0(2) and later, not supported on ASA 100V platform

* Does not support multiple context mode

* Only through-the-box traffic is detected.

* TCP connection attempts that are reset by the targeted server
  are not counted as a SYN attack or scanning threat.

Basic Threat Detection
======================

This component monitors the rates that packets are dropped by the ASA for a
number of reasons, such as ACL drop, Bad Packets, Connection Limits, ICMP
attacks, etc.

The rates at which packets are dropped is measured over a configured time
period, called Average Rate Interval (ARI) which can be from 600 seconds
to 30 days.  If the number of events that occur within the ARI exceeds
the configured rate threshold it is considered the event a threat.

An aditional smaller time period is also monitored, call the burst rate
interval (BRI). If the calculated value exceeds this threshold it is also
considered an adttack.

Basic threat detection is enabled by default on the ASA running
versions 8.0(2) and later.

When a threat is deteced, the ASA simply logs an event with ID
ASA-4-733100  to alert the administrator.  No action is taken
to stop the offending traffic.


Advanced Threat Detection
=========================

Advanced Threat Detection enables the tracking of more granula objects. This
tracking can be done for host IPs, ports, protocols, ACLs and servers protected
by TCP intercept.  By default it is only enabled for ACL statistics.

Multiple time periods are tracked; 20 minutes, 1 hour, 8 hours and 24 hours.
These time periods are not configurable but the number of periods that
are tracked per obbject can be adjusted.

When TCP intercept is configured, Threat Detection can keep track of the top
10 serers which considered to be under attack.

Just like with Basic Threat Detection, the ASA only sends a syslog using
IDs ASA-4-733104 and ASA-4-733105 which are triggered for average and
burst rates respectively.  No other action is taken.

Advanced Threat Detection is also used by ASDM to generate the graphs on the
firewall dashboard.

Scanning Threat Detection
=========================

Scanning Threat Detection build o the concept of Basic Threat Detection, making
use of the same rate-interval, average rate (ARI) and burst rate (BRI) settings.

The scanning detection however maintains a database of atttacker and target IP
addresses that can provide more context around the hosts involved in the scan.

Only traffic that is received by the target host/subnet is considered by this
feature.  Basic Threat Detection can still tricker if the traffic is dropped
by an ACL.

One of the benefits of scanning detection is that it can optionally
react by shunning the attacker IP.  This means that it is the only
threat detection feature which can activelly afffect connections
through the ASA.

When an attack is detected, ASA-4-733101 is logged.  If the shun
feature is enabled, ASA-4-733102 is logged to alert he administrator.

Configuration
=============

.. rubric:: Basic Threat Detection

To enable basic threat detection, use the command:

.. code-block::

  threat-detection basic-threat

A number of 'rates' can be viewed using the command:

.. code-block::

  show run all threat-detection

In order to reconfigure these rates, simply reconfigure the
appropriate rate command accountingly such as:

.. code-block::

  threat-detection rate acl-drop rate-interval 1200 average-rate 2500 burst-rate 550

.. rubric:: Advanced Threat Detection

Use the below command to enable Advanced Threat Detection:

.. code-block::

  threat-detecion statistics

This setting is available for acccess-list, host, port, protocol and
tcp-intercept.

Custom rates can also be configured as wel Basic threat Detection.

.. rubric:: Scanning Threat Detection

To enable Scanning Threat Detection:

.. code-block::

  threat-detection scanning-threat

As with Basic Threat detection the rates can be reconfigured using commands
prefixed with:

.. code-block::

  threat-detection rate scanning-threat ....

To enable shunning of a suspect attacker and configure how long
they should be shunned use:

.. code-block::

  threat-detection scanning-threat shun duration <seconds>

IP addresses and object-group can be set as exceptions to shunning
by specifying them as follows:

.. code-block::

  threat-detection scanning-threat shun except ip-address <ip> <mask>
  threat-detection scanning-threat shun except object-group <objgrp-name>
