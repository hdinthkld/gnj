################################
ASA (Pre-8.3) AnyConnect SSL VPN
################################

.. contents::
   :depth: 3

Introduction
============

.. todo:: Write up the purpose of this chapter


Configuration
=============

Pre-requisites
--------------

.. rubric:: Client-side Configuration

#. User is able to reach the ASA Web Portal
#. Java or ActiveX for portal-based client installation and/or posture checking

.. rubric:: ASA Existing Configuration

#. Ensure all interfaces are configured
#. Ensure necessary routing (static or dynamic) is in place
#. Setup Management (AAA, SSH, HTTP) as required


Configuration Steps
-------------------

.. rubric:: Summary

#. Define define name and domain
#. Generate Encryption Keys
#. Enable WebVPN on appropriate interfaces
#. Define the IP addresses to be assigned to VPN clients
#. Define any necessary NAT exemptions
#. Define Group Policy
#. Define Connection Profile
#. Setup local users (if required)

.. rubric:: Pre-load software packages onto ASA flash

This can be uploaded via ADSM alternatively the ASA can download the package with following command:

::

  copy ftp://<ftp-url>/<file-path> flash:

.. todo ::  

   Verify exact syntax


.. rubric::  Setup hostname and domain name on device

::

  hostname <hostname>
  domain-name <dns-domain>

.. rubric:: Generate Encryption Key

::

  crypto key generate rsa modulus <bit-size>


.. rubric:: Enable Webvpn on the user facing interface

::

  webvpn
    enable <if-name>

    ! Specify AnyConnect package to make available for installation
    svc image <ac-pkg-path>

    ! Show list of available groups at login
    tunnel-group-list enable

.. rubric:: Create usernames (if required)

::

  username <username> <password>


.. rubric:: Create the local IP pool

::

  ip local pool <ip-pool-name> <start-ip>-<end-ip>

.. rubric:: Define any NAT Excemptions

::

  access-list <nat-exemption-acl-name> permit ip <internal-net> <internal-mask> <ip-pool-net> <ip-pool-mask>
  nat (<high-security-interface>) 0 access-list <nat-exemption-acl-name>

.. rubric:: Define the group policy

::
  
  group-policy <gp-policy-name> internal
  group-policy <gp-policy-name> attributes
    ! Set allowed protocols (Clientless and AnyConnect client)
    vpn-tunnel-protocol webvpn svc

    ! Set not to remove AnyConnect client after installation
    webvpn
      svc keep-installer installed
      

.. rubric:: Define the connection profile

::

  tunnel-group <tg-name> type remote-access
  tunnel-group <tg-name> general-attributes
    address-pool <ip-pool-name>
    default-group policy <gp-policy-name>
  tunnel-group <tg-name> webvpn-attributes
    ! Set User Friendly name for the group
    group-alias <tg-alias-name> enable


Verification
============

.. rubric:: Client testing

#. Login to web browser as user and download client software from WebVPN Portal
#. Ensure connection is established
#. Logoff and ensure that connection can be started direct from AnyConnect client icon

Troubleshooting
===============

.. todo:: Document troubleshooting steps
