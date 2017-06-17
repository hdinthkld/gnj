.. _cisco_csm:

############################
Cisco Security Manager (CSM)
############################

Overview
========


Features
--------


Versions
--------

* Standard, Professional, UCS



Server System Requirements
--------------------------

These requirements are based on CSM 4.14

The server must be running a supported operating system. Windows
2012/2016 (64-bit only)  

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

In order for the Client to communicate with the server TCP ports 443/1741
must be open between the Client and Server.


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


Inventory Management
====================

Managing Upgrades
=================


