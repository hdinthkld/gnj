.. _cisco_rbac:

###############################################
Cisco Role-Based Account Control Implementation
###############################################

Cisco NX-OS
===========

Cisco Nexus switces running the NX-OS software support up to 256 of locally
defined user accounts. These accounts can have the following attributes:

* Username
* Password
* Expiry Date
* User Roles

User accounts are local to the VDC however those accounts with the
`network-admin` or `network-operator` roles can login to default VDC and access
the others using the `switchto vdc` command.


Further Information
===================

For more details refer to the Cisco Nexus Security Configuration Guide [c2]_.
