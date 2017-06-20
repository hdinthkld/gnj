.. _cisco_csm:

############################
Cisco Security Manager (CSM)
############################

Overview
========

CSM is a management tool that enables more complex environments to
efficiently manage their security infrastructure. It helps to ensure
consiste policy enforcement and rapid troubleshooting of security
events with in-built reporting capabilities across the security
deployment.

This is achevied by enabling reuse of security rules and objects to
monitor security threats and minimizae potential errors.  Integrated
end-to-end tools are used to facilitate the consistent enforcement
of policy and to aid in rapid troubleshooting.

Security events from multiple devices are consolidated to help
enable viewinng of real-tme and historical events.

Along with the included features, CSM can also integrate into Cisco Talos
for further insight into potential threats.

Depending on the version purchase CSM can support a wide range of devices,
such as:

* Cisco ASA 5500 and 5500-X

* IPS 4200, 4300 and 4500

* Cisco SR 500 Secure Routers

* Cisco AnyConnect Secure Mobility Client

* Cisco FirePower 4100 series (in CSM 4.11 and above)

CSM is based on a client/server module, the users access to the server
database via a software program installed on their workstation. Some
of the tools available to the administrator are:

* Configuration Manager

* Event Viewer

* Report Manager

* Software Image Upgrade Wizad

A managers dashboard is available via a web based interfaces which gives
a birdseye view of the health, functioning and other major performance
indicators of the network security infrastructure.

An API-Based interface is also available to assist with interating into
other network services such as Compliance and security analysis sytems.

CSM can be deployed in a high-availabity setup either for local HA or
Disaster Recovery.  This solution is based on the Symantec Veritas
Storage Foundation and High Availability Solutions. Note that direct
access to PRSM from Security Manager using SSO is only supported in
HA mode.


Features
--------


The following general features available:

* Policy and Object Management

* Event Management

* Reporting and Troubleshooting

* Image Management

* Health and Performance Monitoring (HPM)

* API Access

For ASA series devices CSM provides centralised management for features such
as Botnet Traffic Filter, Clusterig, syslog forwarding. Support for
the AIP modules is also included. Integration with :ref:`cisco_prsm` is also
included when an ASA-CX device is detected.

For IOS devices, zone-based firewall, TrustSec, content filtering and VPN
(GETVPN, DMVPN, GRE and IPSec) technologies is included.


Versions and Licensing
----------------------

As of June 2017 the latest available release of CSM is 4.14

It is a available in 3 different versions (Standard, Professional and
the UCS server bundle).

The standard version is intended to manage less than 25 devices and
supports the most common devices found in a small-to-medium enterprise
such as ASA 5500 Firewalls, IPS 4200/4300, AnyConnect client as well as
ISR G1 & G2 routers. It does not hower support higher-end devices such
as service modules or modular devices. The number of devices supported
is fixed at time of purchase (beween 5 and 25 devices)

The Professional Version is aimed at medium to large deployments (50 to
Hundreds of devices).  It supports all the features of the standard
vesion as well as support for service modules (ASASM and FWSM) as well
as higher ends devices (6500 switches and 7600 routers). The Northbound
API is also included.  Initial licenses can be purchase as 50, 100 and 250
with additional incremental licenses available the same size blocks.

Both of the above versions require an available hardware platform that
is running at least Windows Server 2008 Enterprise R2 with a recommended
16GB of RAM, 4-core CPU and 500GB HDD.

The final version is the UCS server bundle (UCS C220 M3). It provides
all the same features as Professional with the ability to manage 50 to
150 devices (although additional licens can be purchased in same size
blocks as professional).  The key difference between Professional and UCS
is that hardware is provided as part of the purchase with UCS, Professional
is just the licensed software.


The licenses are permitted onl to be used on a single server. However if
a standby service is made available to support High Availability, a seperate
license is not required if only one server is used at a time. The
Administrator is responsbiel for manually restoring the database from the
active to the standby server on a regular basis.

Where a firewall is operating in multiple context mode, each context will
consume a device license in addition to the parent device.

Any devices that exist in CSM but are not selected for "Manage in Cisco
Security Manager" do not consume a device liense. The same is true for
any devices added to the topology map but that do not appear in the device
invventory.

Server System Requirements
--------------------------

For CSM versions upto 4.8, the server must be running a supported operating
system, currently:
* Windows Server 2008 R2 SP1 Enterprise 64-Bit (CSM 4.7 and below)
* Windows Server 2012 Standard/Datacente 64-bit

For CSM versions 4.9 and above:
* Windows Server 2012 Standard/Datacentre (R1 an R2) 64-bit
* Windows Server 2016 Standard/Datacentre 64-bit

The system should have a Quadcore Xeon 5R6500 series or abbove.

In order to run will all features 16GB of RAM is needed. Services such
as Event Management and Report Management are affected if less memory
is detected.

When less than 8GB of RAM is available the Event Management
and Report Manager are automatically disabled.  Between 8 and 12GB the
two services listed can be turned off manually through the "Tools"
section of the Configuration Manager

A minimum of 100GB operating system Partition is recommended. 150GB is
recommended for the application partition with 7GB being the minimum
required.  Seperate OS and applications are recommended

Client system Requirements
--------------------------

The client must have a CPU with minimum of 2GHzand be running Windows 7
, 8, 10 or Windows Server 2012 as a minimum.

A minimum of 2GB of RAM is required for 32-bit systems, 4GB for 64-bit.

10GB free disk space is required.

The client must have a supported Web Browser (IE 8-11, Firefox 15 and above)

Java is not required to e installed seperately as the Security Manager
client includes an embedded version of Java.

The logged in user account is recommended to have full administrator
privileges.  This is the only supported user account by Cisco.

Client to Server Communications
-------------------------------

In order for the Client to communicate with the server TCP ports 443 (HTTPS)
 / 1741 (HTTP) must be open between the Client and Server.

The client must also be running the same version of the software as the
server.  A prompt will appear requesting to download the update if there
is a mismatch.

Device Management
-----------------

.. rubric:: ASA

The CSM server uses the same interface as ASDM in order to commmunicate
with the firewall, therefore TCP 443 (HTTPS) is required to be open.


.. rubric:: Other Devices

Most other network devices can be connected to using the standard
management services HTTPS (TCP 443), SSH (TCP 22), Telnet (TCP 23)

.. rubric:: Configuration Rollback

TFTP is used to transfer the configuration therefore UDP 69 needs
to be open to/from the CSM server to the device.

.. rubric:: ACS

If CSM is integrating with ACS, TCP 2002 should be open


Managing Users
==============

After then initial installation only 'admin' account exists.  This account
can be used add additional acccounts of the following types:

* Local Account

* ACS Account

* Non-ACS Account

CSM provides a number of pre-defined roles which determines the permissions
for the  the user in question:

* System Admin

* Security Admin

* Security Approver

* Network Admin

* Approver

* Network Operator

* Helpdesk


External References
==================

Cisco Security Manager 4.14 Product Overview

http://www.cisco.com/c/en/us/support/security/security-manager-4-14/model.html

Cisco Security Manager 4.14 User Guide

http://www.cisco.com/c/en/us/td/docs/security/security_management/cisco_security_manager/security_manager/414/user/guide/CSMUserGuide.html


Cisco Security Manager 4.14 Installation Guide

http://www.cisco.com/c/en/us/td/docs/security/security_management/cisco_security_manager/security_manager/414/installation/guide/IG.html
