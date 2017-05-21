###################################################
ASA (9.x) SSL AnyConnect Configuration via ASDM
###################################################

Introduction
=============

.. Todo: write introduction to chapter

Configuration
=============

.. rubric:: Pre-requisite configration on ASA

#. Ensure all interfaces are configured
#. Ensure necessary routing (static or dynamic) is in place
#. Setup Management (SSH, HTTP) as required

.. rubric:: Steps Needed

#. Save AnyConnect package to ASA flash
#. Set hostname and domain name
#. Generate Encryption Keys
#. Enable WebVPN
#. Setup local users or external authentication
#. Setup IP Local Pool (Optional if using DHCP)
#. Define filter policy (Optional)
#. Define Split Tunnel Policy (Optional)
#. Define Group Policy
#. Define Connection Profile

.. note:: The above steps can be completed manually as documented below or via the SSL VPN Wizard

.. rubric:: Save AnyConnect Package to ASA Flash

.. todo:: Document methods of uploading to flash

.. rubric:: Set hostname and domain name

In ASDM Navigate to:

:menuselection:`Configuration --> Device Setup`

On the CLI this can be setup as follow:
::

  hostname <hostname>
  domain-name <domain>

.. rubric:: Generate encryption key

::

  crypto key generate rsa modulus <bit-size>

.. rubric:: Enable AnyConnect on the appropriate interfaces

::

  webvpn
    enable <ifname>
    anyconnect image <pkg-path>
    anyconnect enable


.. rubric:: Setup Local Users (Optional)

.. rubric:: Setup external authentication server (Optional)

.. rubric:: Define IP Address Pool

::

  ip local pool <name> <start-ip>-<end-ip>

.. rubric:: Define Group Policies

::

  group-policy <gp-name> internal
  group-policy <gp-name> attributes
    vpn-tunnel-protocol ssl-clientless ssl-client
    webvpn
      anyconnect ask enable
      anyconnect keep-installer installed

.. rubric:: Define Connection Profile

::

  tunnel-group <name> type webvpn
  !
  tunnel-group <name> general-attributes
    default-group-policy <gp-name>
    address-pool <ip-pool-name>
  !
  tunnel-group <name> webvpn-attributes
    group-alias <alias-name>
