################################
ASA (Pre-8.3) SSL Clientless VPN
################################

Introduction
============

Client Requirements
-------------------

#. Only browser required for ClientLess SSL VPN
#. Java is required to operate as a thin-client

Configuration
=============

.. rubric:: Pre-requisite ASA Configuration

#. Ensure all interfaces are configured
#. Ensure necessary routing (static or dynamic) is in place
#. Setup Management (SSH, HTTP) as required


.. rubric:: Steps Needed

#. Define define name and domain
#. Generate Encryption Keys
#. Enable WebVPN on appropriate interfaces
#. Define Port Forwarding List (optional)
#. Define Group Policy
#. Define Connection Profile
#. Setup local users (Optional)
#. Import Plugins onto ASA (Optional)
#. Configure Smart Tunnel (Optional)

Setup hostname and domain name on device
----------------------------------------

::

  hostname <hostname>
  domain-name <dns-domain>

Generate Encryption Key
-----------------------

::

  crypto key generate rsa modulus <bit-size>


Enable Webvpn on the user facing interface
------------------------------------------

::
  webvpn
    enable <if-name>

Setup AAA Servers
-----------------

.. note:: Ensure the AAA Server is configured to allow the ASA (client) to
          request authentication, same key should be used on both devices.

::

  aaa-server <aaa-group-name> protocol {tacacs+|radius}
    aaa-server <aaa-svr-name> (<ifname>) host <aaa-svr-ip>
      timeout <seconds>
      key <aaa-key>



Create usernames (if required)
------------------------------

::
  username <username> <password>


.. rubric:: Create Port forwarding rules


::
  webvpn
    port-forward <pf-list-name> <local-port> <remote-ip> <remote-port>

.. rubric:: Setup Smart Tunnel

::

  smart-tunnel list <st-list-name> <program-friendly-name> <prog-executable>


.. rubric:: Define the group policy

::

  group-policy <gp-policy-name> internal
  group-policy <gp-policy-name> attributes
    ! Set allowed protocols (SSL only in this case)
    vpn-tunnel-protocol webvpn

    ! Define the port forwarding for this group
    webvpn
      port-forward name <pf-list-name>
      port-forward enable <pf-list-name>

      ! Set port forwarding to autostart
      port-forward auto-start <pf-list-name>

      ! Define what programs to have their traffic sent via the tunnel (Smart-Tunnel)
      smart-tunnel enable <st-list-name>


.. rubric:: Define the connection profile

::

  tunnel-group <tg-name> type webvpn
  !
  tunnel-group <tg-name> general-attributes
    default-group policy <gp-policy-name>
    authentication-server-group <aaa-group-name>
  !
  tunnel-group <tg-name> webvpn-attributes
    ! Set User Friendly name for the group
    group-alias <tg-alias-name> enable

.. rubric:: Import SSH/TELNET plugins from external file store

.. todo: Document how to import and setup manually

Java Plugins are provided as .jar files and are available for the following protocols:
  * SSH/Telnet
  * RDP
  * Citrix ICA

::

  import webvpn plug-in protocol <protocol> <url-of-jar-file>

Verification
============

.. rubric:: Test AAA Authentication

::

  test aaa-server authentication <aaa-group-name> host <aaa-svr-ip> username <username> password <password>
