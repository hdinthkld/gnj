.. _cisco_stormcontrol:

###################
Cisco Storm Control
###################

Overview
########

A traffic storm ocurs when packets are flooding the LAN which results in excessive trafic and degraded network performance.

The traffic storm control feature prvents LAN ports from bein distrupted by broadcast, multicast or unicast trafffic storms on physical interface.

Traffic Storm control is also sometimes called traffic suppression and it monitors incoming traffic levels of a 1 second interval.  During this interval the trafffic level is comparred with the storm control level which has been configured.  This level is a percentage of thhe total available bandwidth on the port.

Each port has a single trafic storm control level that is used for all types of traffic (broadcast, multicast and unicast).

By default when traffic exceeds the configured level the traffic is dropped until the trafffic storm control interval ends. It is also possible to simply
send an SNMP trap when a traffic storm occurs.

When both multicast and broadcast storm control is enabled if broadcast and/or multicast traffic exceeds the configured level, both types of traffic are dropped.

Configuration
##############

Configuration is done at the physical interface or Port Channel level. In the case of port channel interfaces, do not configure storm control on the individual member ports.

.. code-block:: none

  interface <type/slot>
    storm-control broadcast level <level>

    ! Only supported on Gigabit Ethernet interfaces
    storm-control multicast level <level>

    ! Ony supported on Gigabit Ethernet interfaces
    storm-control unicast level <level>

Monitoring and Troubleshooting
##############################

.. code-block:: none

  show interfaces switchport

  show interfaces counters storm-control


Further Reading
###############

Release 15.1SY Supervisor Engine 2T Software Configuration Guide [c8]_
