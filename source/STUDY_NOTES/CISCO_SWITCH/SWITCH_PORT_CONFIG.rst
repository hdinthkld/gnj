****************************************
Cisco - Switch Port Configuration
****************************************

.. _ccnp_switch_switch_port_config:

Ethernet Overview
=================

- Determining Bandwidth Requirements

  * Types of applications used
  * Traffic flow in use
  * Size of user community

- Links between access, distribution and core should be scaled to match required load
- Ethernet based on IEEE 802.3 standard
- A shared medium becoming both a collision and broadcast domain
- Based on Carrier Sense Multiple Access with Collision Detection (CSMA/CD)
- More "Crowded" segments likely to suffer more collisions and therefore operate ineffectively
- switching eliminates the collection problem by dividing a single segment into multiple segments, each port is a unique collision domain
- Hosts/stations can operate in full duplex when using switched, effectively doubling bandwidth.

Scaling Ethernet
================

- Originally 10Mbps per segment
- 100Mbps, 1Gbps, 10Gbps, 40Gbps and 160Gbps standard exist

- Fast Ethernet (100Mbps)

  * Supported on UTP, STP and fibre-optic cables
  * UTP/STP limited to 100 Metres
  * 62.5/125 Multimode fibre (MMF) supported 400 Metres (half-duplex) or 2000M (Full Duplex)
  * Single Mode fibre (SMF) max 10KM
  * Can be bundled into etherchannels to increase bandwidth

- Gigabit Ethernet (1Gbps)

  * Uses same 802.3 frame format
  * Physical later modified to increase transmission speeds
  * Merges 802.3 and ANSI X3T11 Fibre Channel Standards
  * Supported on STP up to 2M (1000Base-CX)
  * UTP Cat 5 upto 100M (1000Base-T)
  * Fibre Optic

    * 1000Base-SX MMF 62.5 (275M) or 50 (550M)
    * 1000Base-LX/LH MMF 62.5 (550M) or 50 (550M), SMF 9 (10KM)
    * 1000Base-ZX SMF 9 (70KM), 8 (100KM)

  * "Gigabit over copper" switch ports can operate at 10/100Mbps or 1Gbps
  * Gigabit Ethernet offers up to 16Gbps (Full Duplex)

- 10-Gigabit Ethernet (10Gbps)

  * Layer 2 details unchanged
  * Uses different transceivers - Physical Mediia Dependant (PMD) Interfaces
  * LAN PHY - Connects switches to Campus (Core) Layer
  * WAN PHY - Connects existing SONET/SDH, typically found in Metropolitan Area Networks (MAN)
  * PMD Interfaces use "10GBASE" prefix
  * Runs over MMF (33M - 330M)
  * Runs over SMF (10KM)
  * Copper: CX4  with Infiniband (15M)

  * Transceiver Types
  
    * Wavelength - S = Short, L = Long, E = Extra Long
    * PHY Type - R = LAN PHY, W = WAN PHY
    * LX4/LW4 Long Wave length, X/W indicate the coding used, 4 is number of wavelengths
    * WWDM = Wide Wavelength Division Multiplexing

- Beyond 10-Gigabit Ethernet

  * 40Gbps bonds 4 x 10Gbps using a Quad SFP+ Module (QSFP+)
  * 100Gbps bonds multiple channels/lanes
  * 40/160Gbps defined in 802.3ba Standard

Duplex Operation Over Ethernet
==============================

 - Max throughput only possible when one device is connected to switch port
 - Switch ports are capabble of negotiating both speed and duplex, both ports muust be configured 
   for auto-negotiation
 - Link speed is determined by electrical signaling, highest common speed is used
 - Duplex mode involves and exchange of information. Both ports must be auto otherwise half-duplex is the default
 - Errors on a link will occur when there is a duplex mismatch
 - Cisco recommendation is to use manual setings in order to avoid unusable connections

 Connecting Switches And Devices
 ===============================

- Catalyst switches 10/100/1000 auto sensing ports using RJ-45 connections with UTP cable
- UTP cabling uses 4 pairs (Pins 1/2, 3/6, 4/5, 7/8) to connect straight through to other end
- Gigabit provides SFP modules

  * Hot swappable
  * LC/MT-RJ Fibre Optic, RJ45 UTP Copper connectors

- 10 Gigabit Ethernet uses X2 and SFP+ modules

Configuring Switch ports
========================

- Performed through global configuration mode

**Selecting a single switch port**

::

  interface <type> <memmber>/<module>/<number>

**Selecting multiple ports**

::

  interface range <name> , <name-2> [, <name-x>]
    <commands>

  or 

  interface range <name> - <name>
    <commands>

**Using Interface macros**

::

  define interface-range <name> [ , <name> ] [ - <name>]
  interface range macro <name>
    <commands>


**Adding Comments to interfaces**

::

  interface <name>
    description <one-line-string>


**Manually setting port speed**

::

  interface <name>
    speed {10|100|1000|auto}


**Manually setting duplex mode**

::

  interface <name>
   duplex {auto|full|half}

.. _switch_errdisable:

Managing Error Conditions On Switch Ports
=========================================

General notes on error Conditions
---------------------------------

- Network management applications can poll devices to check for errors
- Catalyst switches can detect errors an take action automatically
- By default switches will detect errors on every port for all causes
- By default switch ports must be manually shutdown then restored in order to recover
- Ports are disabled for 300 seconds by default if automatically error recovery is enabled

**Tune trigger clauses globally**

::

  [no] errdisable detect cause {all | <cause>}

**Enable Auto Recovery**

::

  errdisable recovery cause {all | <cause>}

**Change recovery timer**

::

  errdisable recovery interval <seconds>

Troubleshooting Port Connectivity
=================================

**Check Port State - Up/Up is expected for normal operation**

::

  show interfaces [<name>]

**Get Summary of all switch port states**

::

  show interface status

**Show ports in error disabled states**

::

  show interface status err-disabled

Checking for speed/duplex mismatches
------------------------------------

- Check for error count greater than 0
- "Runts" are packets truncated before being fully received
- Input" errors usually show
- Check if running at half-duplex, indicating unsuccessful auto-negotiation

Discovering Connected Devices
=============================


.. _switch_cdp:

Cisco Discovery Protocol
------------------------

- Automated method for devices to advertise prescense to directly connected neighbors
- Cisco propietary feature
- One-Way, no response expected
- Sent at the Data Link Layer (Layer 2) every 60 seconds

- Information advertised

  * Hostname
  * Connected Interface
  * Device Capabilities
  * Hardware Platform
  * Software Version
  * Duplex Mode
  * Power Info
  * Management IP(s)

- Cisco routers and switches have CDP enabled by default

**See advertised neighbours**

::

  show cdp neighbors [<interface-name>] [detail]

**Disable/Enable CDP globally on the switch**

::

  [no] cdp run

**Disable/Enable CDP per interface**

::

  [no] cdp enable

**View CDP info for named device**

::

  show cdp entry <name>


.. _switch_lldp:

Link Layer Discovery Protocol
-----------------------------

- Based on IEEE 802.1ab
- Supported By Multiple Vendors
- Extendable through Type-Length-Values (TLV) structures
- Specific Media Endpoint TLVs (LLDP-MED)
- Must use either bbasic LLDP or LLDP-MED per interface, not both
- By default globally disabled

**Globally disable/enable LLDP**

::

  [no] lldp run

**Display LLDP Advertisements Received**

::

  show lldp neighbors [<interface-name>]

**Disable/Enable LLDP Per interface**

::

  [no] lldp {receive | transmit}

Using Power Over Ethernet (POE)
===============================

- PoE can be used to provide power to IP phones and wireless access points (WAPs)
- Provide a means of centralised power management, unlike using individual wall warts
- PoE can be managed, monitored and offerred only to known devices
- Best Practice: Connect switch to UPS so power can be maintained in the event of a power outage

How PoE Works
-------------

- Switches must be rated to supply power to external devices
- PoE methods

  * Cisco Inlne Power (ILP) - Cisco proprietary upto 7W
  * IEEE 802.3AF (PoE) up to 15.4W
  * IEEE 802.3AT (PoE+) up to 25.5W
  * Cisco Universal Power (UPoE) - Cisco propertiary up to 60W

Detecting A Powered Device
--------------------------

- Power is disabled when port is down
- Small voltage is sent across transmit/receive pairs
- Resistance is measured and if detected a device must be present
- Resistance values determine a specific power clauses

  * Class 0 - Default (15.4W)
  * Class 1 - 4.0W
  * Class 2 - 7.0W
  * Class 3 - 15.4W
  * Class 4 (802.3AT) - Upto 30W

- Class 0 used when device/switch does not attempt discovery
- Max of 15.4W is usually offered
- CDP/LLDP can be used to request power up to 30W
- Special TLVs used with CDP/LLDP for UPoE (Catalyst 4500 only)


Configuring PoE
---------------

- General Notes

  * By default each port auto-detects if power is needed
  * If a device requests more power than configued, a log message is generated and port 
    remains in non-connected state

**Configure Power offered on a port**

::

  power inline auto [max <milliwatts>]
  power inline static
  power inline never


Verifying PoE
-------------

** Monitor power available/used/total **

::

  show power inline [module <member>] [detail]
  show power inline [<interface-name>] [detail]