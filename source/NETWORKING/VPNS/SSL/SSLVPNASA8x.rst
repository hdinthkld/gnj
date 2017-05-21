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

Pre-requisites
--------------

ASA Existing Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Ensure all interfaces are configured
#. Ensure necessary routing (static or dynamic) is in place
#. Setup Management (AAA, SSH, HTTP) as required


Key Configuration Steps
-----------------------

#. Define define name and domain
#. Generate Encryption Keys
#. Enable WebVPN on appropriate interfaces
#. Define Port Forwarding List (optional)
#. Define Group Policy
#. Define Connection Profile
#. Setup local users (if required)

Setup hostname and domain name on device
----------------------------------------

::

  hostname <hostname>
  domain-name <dns-domain>

Generate Encryption Key
-----------------------

::

  crypto key generate rsa modules <bit-size>


Enable Webvpn on the user facing interface
------------------------------------------

::

  webvpn
    enable <if-name>


Create usernames (if required)
------------------------------

::

  username <username> <password>


Create Port forwarding rules
----------------------------

::

  webvpn
    port-forward <pf-list-name> <local-port> <remote-ip> <remote-port>

Define the group policy
-----------------------

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
      

Define the connection profile
------------------------------
::

  tunnel-group <tg-name> type webvpn
  tunnel-group <tg-name> general-attributes
    default-group policy <gp-policy-name>
  tunnel-group <tg-name> webvpn-attributes
    ! Set User Friendly name for the group
    group-alias <tg-alias-name> enable


Verification
============

Client Verification
-------------------

* Login as the user with a web browser

ASA Verification
----------------

* Verify the number of users connected

::

  show vpn-sessiondb

* View details for a specific user

::

  show vpn-sessiondb user <username> detail