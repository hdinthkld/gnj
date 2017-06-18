.. _cisco_prime_infrastructure:

###############################
Cisco Prime Infrastructure (PI)
###############################

Overview
========

.. note:: Cisco Prime Infrastructure should not be confused with :ref:`cisco_prsm`.
          PRSM was both a built-in tool and standalone server used to manage the
          EOL'd Cisco ASA-CX Module which has now been replaced by Cisco Firepower and it's
          manageable tool, the FireSight Management Centre (FMC).

Prime Infrastructure is Cisco's vision for "One Management, One Assurance and
One Network".

It is said to provide a comprehensive solution to manage, visualise and monitor
thhe network from a single graphical interface.  It provides the following
general features:

* Lifecycle Management
* Assurance Visibility
* Troubleshooting capabilities network wide

It can integrate with other tools such as:

* :ref:`Cisco Identity Services Engine (ISE) <cisco_ise>`
* :ref:`Cisco Mobility Services Engine (MSE) <cisco_mse>`
* :ref:`Cisco ClearAir <cisco_cleanair>`

New versions include even more features suh as the Configuration Compliance
Engine which enables operators to provide the baseline configuration upon
which to audit the network devices and the configurations stored in the
archive. This helps identify devices that  are out of compliance for EoL/EoS
monitoring other compliance frameworks, such as PCI or HIPAA.

Features
--------

* Single-pane-of-glass management
* Simplified deployment of Cisco capabilities
* Deep Application Visibility
* Comprehensive coverage of enterprise mobility
* Unified assurance across network and computer
* Centralised Visibility of distrubted networks

As of version 3.0, PI includes:

* Platform Enhancements
* Wireless Management
* Routing IWAN Management
* Data Centre Management
* APIC-EM Integration

Versions
--------

As of June 2017, Prime Infrastructure 3.1 is the latest version.

Prime Infrastructure is available as a Virtual or Physical appliance in the
following versions:

* Express (Upto 500 devices)
* Express-Plus (Upto 3000 devices)
* Standard (Upto 10000 devices)
* Professional (Upto 14000 devices)
* Hardware Appliance Gen 2 (Upto 24000 devices)

Licensing
---------

Licensing for Prime Infratructure is divided into base license and
corresponding features licenses. The maximum number of license files that
can be added is 25.

When performing a first time installation, the lifecycle and assurance features
are accessible using a built-in evaluation license which is valid for 60 days
and 100 devices.

Six types of licenses are available:

* Base (Required)

* Lifecycle (Total number of devices under Prime Infrastructure  Management)

* Assurance (Total Number of NetFlow devices under management). Also
  requires a Lifecycle license.

* Collector (Total Number of NetFlow data flows per second that can be
  processed). Also requires an Assurance license.

* Data Centre (Number of Blade Servers being managed by UCS devices, not
  including Nexus switches or MDS devices)

* Data Centre Hypervisor (Total number of hosts on VCentre being managed)

Lifecycle, Assurance and Data Centre are available for both evaluation and
permenant licensing.  All others do not have an explicit evaluation version.

An additional licensed feature is for Prime Infrastructure Operations Centre
which enables support for managing multiple instance of PI from a single
instance. A maximum of 10 managed instances is supported.

System Requirements
-------------------

Managing Upgrades
=================

Prime Infrastructure provides facilities to assist with planning, scheduling
and downloding and monitoring the software image updates.

The images are stored on the Prime infrastructure acording to the image type
and version.

In order manage software images the devices must be cofigured with Telnet/SSH
credentials as well as SNMP read-write community strings.

Note that only SFTP, SCP and FTP are supported for image distribution. TFTP is
not supported.

Prime can also import the software images from the devies.  The same protocols
as above as wwell as TFTP are supported, SFTP or SCP is recommended.

Software Images are managed via the Inventory under:

:menuselection:`Inventory --> Device Management --> Software Images`

Prime supports the Cisco devices below for distribution and activation using
the Software Image Management Server:

* WLC
* Nexus 3K, 5K, 7K, 9K
* Cat 4K, 6K, 5760, 3750, 2960
* Catalyst 3650/3850
* ASR 9K
* ISR 1841, 1900, 2800, 2951, 3825, 3845, 890
* ME 3800
* IE 4000
* ASR 1000

Prior to performing upgrade, Prime Infrastructure can run a report to help
determine the prerequisites for new software image deplyment. This will
provide information on if the device has sufficent, RAM, Flash storage as well
as if the image is suitabble for the device.

To run this report go to the same menu item as above and select
:guilabel:`Upgrade Analysis`.
