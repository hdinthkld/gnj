#################################
Cisco ASA Firewall Virtualisation
#################################

Overview
########

As of ASA version 7.0(1) it has been possible to make a single ASA device to act
as multiple standalone firewalls.  Each firewall behaves independently with its own configuration.

It is possible to setup independent administrative access so one administrator
can manage one context but not another one.

Cisco called this individual firewalls *Security Contexts* for which there are 3 different types:

* System Execution space (system context)
* Admin context
* One o more user contexts (customer contexts)


.. rubric:: System Execution Space

The system context  does not have any layer 2/3 interfaces or network settings. It is used purely for the purpose of managing other security contexts. The
three most important settings for each context are:

* Context Name
* Location of the contexts startup configuration
* Interface Allocation

Other settings such as interface, NTP, Resource Management, failover and boot parameters are also configuered in the system context.

It is possible to retrieve the configuration files for the security contexts via
TFTP, FTP, HTTP or HTTPS.

.. rubric:: Admin context

The admin contet provides connectivity to the network resources such as AAA and syslog servers.  It is recommended that the management interface of the security appliance be on the admin context.

If an administrator has access to the admin context, he can also jump to all the other security contexts.

The admin context is used by the system context for any actions that require Layer 2 or Layer 3 functionality. Therefore the admin context must be created before you define any other contexts.

When a firewall is changed from a single device to multiple contexts, the  network related config is moved into the admin context. By default this context is name 'admin'.

Although the admin context acts just like a regular user context it is not recommended to do so due to the it's importants to the overall system.

.. rubric:: User Context

Each user context holds the specific configuration for that context.  Such as:

* IPS
* Dynamic Routing (EIGRP and OSPFv3 as of 9.0.1)
* Packet Filtering
* NAT
* Site-To-Site VPN (as of 9.0.1)
* IPv6
* Device management

Licensing
=========

Depending on the model of the device there is a maximum number of security contexts is limited. For example the 5510/5515-X supports up too 5 contexts where as the 5585-X with an SSP-20 supports upto 250 which is the maximum for all platforms.

.. note:: The  ASA 5505 does not support running any virtual firewalls.

Implementation
##############

In order to identify which context the ASA needs to direct packets to it can
use one of two methods:

* Non-Shared Interface (Physical or Virtual)
* Shared Interface

With the shared interface method, if a non-unique MAC address is used (i.e one shared across all contexts) then NAT rules must be setup on each security context so that the ASA can learn about the subnets located behind each context.

It is recommended that in a shared interface environment that either a MAC address is assigned manually to each context or the system is allowed to automatically generate a system MAC when the context is created (Default in 8.5.1 and above).

Resource Management
===================

To prevent one firewall from consuming all resources on the system it is possible to limit each contexts use of the following:

* Number of concurrent address translations (xlates)
* Number of concurrent connections
* Number of concurrent hosts
* Inspection rate
* Conn Rate
* Syslog Rate
* Telnet, ASDM and SSH Management Systems
* Routes
* MAC Addresses (Tranparent mode)

By default no limits are imposed on the contexts. It is also possible to oversubscribe the available resources.

Resource Management is configure through a Resource Class



Configuration Steps
====================

#. Change to Multiple Mode
#. Setup the system space
#. Configure Interfaces
#. Specify configuration URL
#. Configure Admin context
#. Configure User context
#. Manage the security contexts (optional)
#. Setup Resource Management (optional)

.. rubric:: Change to Multiple Mode

.. code-block:: none

  mode multiple

When entered the ASA will prompt to reload the device. After the reboot the mode
can be configured by:

.. code-block:: none

  show mode

.. rubric:: Setup System space

The original management address will be moved to the admin context an this
is where the administator starts after logging in. To ensure the system context use:

.. code-block:: none

  changeto system


Monitoring and troubleshooting
##############################

The following commands can be used to assist with troubleshooting any issues:

.. code-block:: none

  show mode

  show context

  show admin-context

  show cpu usage context all

  show asp drop
