.. _cisco_prsm:

###################################
Cisco Prime Security Manager (PRSM)
###################################

Overview
========


.. note:: Cisco Prime Security Manager is no longer a current Cisco product and along with
          the ASA-CX module has been replaced by Cisco FirePower and the 
          :ref:`Cisco FirePower Management Console (FMC) <cisco_fmc>`. It should also not be 
          confused with  :ref:`cisco_prime_infrastructure` which is still a current product.

PRSM provides a centralised, simple and scaleable tool to manage Cisco ASA 5500-X appliances
and ASA-CX modules. It provides context-aware capabilities for Application Visibility and
Control (AVC), Web Security Essentials (WSE) and Intrusion Prevention Systems (IPS).

Available as both a built-in user interface (ASA-CX module) as well as standalone
server (Physicl Appliance or ESXi VM) it provides for both single and multi-device
management with a consistent administration experience.

Features
--------

* Network Visibiliy
* Granular Application, User and Device Control
* Flexibility Management Architecture


Versions and Licensing
----------------------

Prime Security Manager is only available in one version but can be purchased as either
a Software Virtual Appliance or Physical Appliance.

Both form factors can be be initial purchased with licensing for managing 5,10,25,50
and 100 devices. After tis initial purchased further management licenses can be 
managed in he same size blocks.


The latest and last version of PRSM available is 9.3

System Requirements
-------------------

Inventory Management
====================

* Health Monitor

Managing Upgrades
=================



Reporting
=========

* Traffic Summary
* Applications
* Users
* Endpoint
* URL
* Device
* Threats


Flexible Management Architecture
================================

A consistent management interface is provided regardless of if a single device or
multiple devices are being managed.  

When managing multiple devices all accces requests are redirected to the primary manager to
ensure efficient, centralised control.

When migrating from the management of a single device to multiple devices existing network
definitions can be imported from other ASA security devices and used to contruct new
policy rules.

After migration to multi-device mode, policies can be shared across multiple firewalls
again ensuring consistent policy enforcement.

Limited configuration is available in order to manage the ASA itself through Firewall NAT and
events.   Commands can also be previewed prior to deployment.

ASA-CX Failover
==============

The ASA-CX modules do not exchange configuration or onnection state information with
their peer. Failover must be configured independently of the parent ASA. PRSM
Multiple Device Mode can be used in order for the configuration to be kept in sync but
does not allow connection state information to be communicated.

When a failover event occurs the newly active ASA will only redirect new connections
through the CX services.  Existing connections will not be redirected.

Managing ASA-CX in Multiple Device Mode
=======================================

In order to manage the ASA-CX in PRSM Multiple-Device Mode the CX module needs to be
put in Managed Mode.  Once this is done you cannot configure the CX module through it's
web interface, all configuration and monitoring must be done via the PRSM interface.

The CLI is still available for troubleshootting and basic configuration (IP addressing, 
DNS, NTP and passwords).

If an ASA that contains a CX is added to the Inventory of PRSM the CX module will also be
addeded. The device is placed in managed  mode once changes are committed.

Features of Managed Mode
------------------------

* Unified Monitoring of multiple devices

* Shared Objects and Policies

* Universal Policies (9.2+) to ensure consistent top an bottom-evel policy enforcement

* Deployment Manager for scheduled configuration deployment

* Centralised License Management

* CX Failover Support to ensure configuration parity

* ASA Management limited prior to 9.2 but now includes unified object,
  interfaces, ACLs, NAT, logging and failover suppot.

Implications of Managed Mode
----------------------------

* All discovered policy elements are added to the PRSM database. For anything that
  is not discovered (e.g. Signatue Updates and Network Particiption), the settings 
  are replaced with the settings currently defined in the PRSM database.

* A CX device can only be managed by a single PRSM server and maintains awareness of
  which server it is being managed by.

* Events from the CX device are forwarded autoamtically to the PRSM server

* PRSM automatically collects database data from all managed devices

* The CX device's web interface will indicate that the device is in Managed Mode along
  with a lin to the PRSM server that is managing the device.

* PRSM uses SSL (HTTPS) in order to communicate with the ASA, the HTTP server must
  therefore be enabled so that PRSM can connect to it. It must use an IPv4, not IPv6
  for management.

* PRSM can only manage an ASA in single-context routed or transparent-mode. If the
  ASA is configuredi in multiple-context mode the CX module can still be managed.

* Active/Standby failover on the ASA is supported, Active/Active is not.

* ASA CX in monitor-only mode is not supported



