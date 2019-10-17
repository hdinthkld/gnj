***************************************
Cisco - Understanding High Availability
***************************************

Leveraging Logical Switches
===========================

- Switches as each network layer should be deployed in pairs for redundancy
- Traditional devices provided redudancy but don't allow all links to be used
- Redudancy only provided at core and distribution layers, not access
- Logical switches permits grouping the redudant links into an etherchannel, no blocked links
- Two approaches to logical switches

  * StackWise
  * Virtual Switching System (VSS)

.. _switch_stackwise:

StackWise
---------

- StackWise and StackWise Plus options available
- Supported on Catalyst 3750-E, 3750-X and 3850
- Uses special "stacking" cables deployed in a daisy-chain to form a ring
- Traffic is carried bi-directional across the stacking switch fabric
- Switches can be added/removed as long the ring is kept intact in one or both directions
- One switch is appointed as the master to perform management functions
- Single management IP is assigned to the entire switch stack

Virtual Switching System
------------------------

- Two identical Chassis switches working as a single switch
- Supported on Catalyst 4500R, 6500 and 6500 series switches
- Referred to as a "VSS Pair"
- Multiple interfaces connect the switches called the "Virtual Switch Link" (VSL)


SuperVisor And Route Processor redundancy
=========================================

- FHRP protocols (VRRP, HSRP, GLBP) provide HA only for the default gateway IP, no assistane for directly connected hosts
- Chassis switch can support multiple supervisors, offering redundancy within the switch itself
- Some switches also have multiple power supplies offering the ability to support dual power feeds in the event of power failure

Redudant Switch supervisors
---------------------------

- First of the two supervisors to boot becomes active, others take on a standby role
- Active supervisor boots to a fully initialised and operational state
- Standby only initialised to a certain level depending on configured/supported modes

  * Route Processor Redudancy (RPR)
  * Route Processor Reudancy Plus (RPR+)
  * Stateful Switch Over (SSO) - Best

- Single Router Mode (SRM) indicates two route processors but one one is active
- Dual Router Mode (DRM) indicates both route processors are active, commonly alongside HSRP
- SRM is not comatile with  RPR/RPR+
- SRM is inherent with SSO, can be seen as "SRM with SSO"

- Route Processor Redundancy (RPR) Mode

  * Supervisor only partially booted
  * Every module in switch is reloaded when active fails
  * Layer 2/Layer 3 not initialised

- Route Processor Redundancy Plus (RPR) Mode

  * Supervisor and Route Engine Initialised
  * No Layer 2/Layer 3 functions started
  * No reload of modules needed when active fails
  * Switch ports retain existing state

- Stateful Switch Over (SSO) Mode

  * Supervisor Fully Booted And Initialised
  * Startup/Running config synchronised
  * Layer 2 information maintained
  * No change to interface states
  * Layer 3 protocols and routing protocol convergence happens upon active failure

Non-Stop Forwarding (NSF)
=========================

- Focuses on quickly rebuilding routing information base (RIB)
- RIB used to generate FIB for CEF
- Router can use NSF to get assistance from other NSF-Aware neighbours
- Avoids waiting on routing protocol convergence
- NSF must be supportd by the routing protocol (BGP, EIGRP, OSPF, IS-IS)
- NSF is Cisco Proprietary

Configuring High Availability
=============================

**Set the redundancy mode for the chassis**

*NOTES:*

- Needs to be configured on both switches the first time it is setup
- To support RPR+ both chassis must have identical IOS otherwise fails back to RPR

::

  redundancy
    mode { rpr | rpr-plus | sso }

**Verify Redundancy Status**

::

  show redundancy status

**Configure Suprvisor Synchronisation**

*NOTE: By default startup and running configurations are sync'd*

::

  redudancy
    main-cpu
      auto-sync standard # default setting
      auto-sync { startup-config | config-register | bootvar }

**Configure NSF For BGP**

::

  router bgp <as>
    bgp graceful-restart

**Configure NSF For EIGRP**

::

   router eigrp <as>
     nsf

**Configure NSF For OSPF**

::

  router ospf <process-id>
    nsf

**Configure NSF For IS-IS**

::

  router isis [<tag>]
    nsf [cisco | ietf ]
    nsf interval [<mins>]
    nsf t3 { manual | [seconds] | <adjacency> }
    nsf interface wait <seconds>
