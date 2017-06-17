###############################################################################
Cisco CCNP - Implementing Cisco Edge Network Security Solutions (300-206 SENSS)
###############################################################################

.. toctree::
   :titlesonly:
   :caption: Contents:

   REFERENCE

.. role:: strikethrough

1.0 Threat Defense (25%)

  1.1 Implement firewall (ASA or IOS depending on which supports the implementation)

  1.1.a Implement ACLs on :ref:`IOS <cisco_iosfw_rules>`, :ref:`ZBFW <cisco_zbfw_rules>`
  and :ref:`ASA <cisco_asafw_rules>`

  :strikethrough:`1.1.b Implement static/dynamic NAT/PAT`

  :strikethrough:`1.1.c Implement object groups`

  1.1.d Describe threat detection features

  1.1.e Implement botnet traffic filtering

  1.1.f Configure application filtering and protocol inspection

  :strikethrough:`1.1.g Describe ASA security contexts`

1.2 Implement Layer 2 Security

  1.2.a Configure DHCP snooping

  1.2.b Describe dynamic ARP inspection

  1.2.c Describe storm control

  1.2.d Configure port security

  1.2.e Describe common Layer 2 threats and attacks and mitigation

  1.2.f Describe MACSec

  1.2.g Configure IP source verification

1.3 Configure device hardening per best practices

  1.3.a Routers

  1.3.b Switches

  1.3.c Firewalls

2.0 Cisco Security Devices GUIs and Secured CLI Management (25%)
  2.1 Implement SSHv2, HTTPS, and SNMPv3 access on the network devices

  2.2 Implement RBAC on the ASA/IOS using CLI and ASDM

  2.3 Describe Cisco Prime Infrastructure

    2.3.a Functions and use cases of Cisco Prime

    2.3.b Device Management

  2.4 Describe Cisco Security Manager (CSM)

    2.4.a Functions and use cases of CSM

    2.4.b Device Management

  :strikethrough:`2.5 Implement Device Managers`

    :strikethrough:`2.5.a Implement ASA firewall features using ASDM`

3.0 Management Services on Cisco Devices (12%)

  3.1 Configure NetFlow exporter on Cisco Routers, Switches, and ASA

  3.2 Implement SNMPv3

  3.2.a Create views, groups, users, authentication, and encryption

  3.3 Implement logging on Cisco Routers, Switches, and ASA using Cisco best practices

  3.4 Implement NTP with authentication on Cisco Routers, Switches, and ASA

  3.5 Describe CDP, DNS, SCP, SFTP, and DHCP

  3.5.a Describe security implications of using CDP on routers and switches

  3.5.b Need for dnssec

4.0 Troubleshooting, Monitoring and Reporting Tools (10%)

4.1 Monitor firewall using analysis of packet tracer, packet capture, and syslog

  :strikethrough:`4.1.a Analyze packet tracer on the firewall using CLI/ASDM`

  :strikethrough:`4.1.b Configure and analyze packet capture using CLI/ASDM`

  :strikethrough:`4.1.c Analyze syslog events generated from ASA`


5.0 Threat Defense Architectures (16%)

  5.1 Design a Firewall Solution

    :strikethrough:`5.1.a High-availability`

    :strikethrough:`5.1.b Basic concepts of security zoning`
    
    :strikethrough:`5.1.c Transparent & Routed Modes`
    
    :strikethrough:`5.1.d Security Contexts`

  5.2 Layer 2 Security Solutions

    5.2.a Implement defenses against MAC, ARP, VLAN hopping, STP, and DHCP rogue attacks

    5.2.b Describe best practices for implementation

    5.2.c Describe how PVLANs can be used to segregate network traffic at Layer 2

6.0 Security Components and Considerations (12%)

  6.1 Describe security operations management architectures

    :ref:`6.1.a Single device manager vs. multi-device manager <asacx_multiple_mode>`

  6.2 Describe Data Center security components and considerations

    6.2.a Virtualization and Cloud security

  6.3 Describe Collaboration security components and considerations

    6.3.a Basic ASA UC Inspection features

  6.4 Describe common IPv6 security considerations

    :ref:`6.4.a Unified IPv6/IPv4 ACL on the ASA <cisco_asa_unified_acl>`
