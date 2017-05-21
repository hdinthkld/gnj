##################################################
ASA (9.x) SSL ClientVPN Configuration via ASDM
##################################################

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

#. Set hostname and domain name
#. Generate Encryption Keys
#. Enable WebVPN
#. Setup local users or external authentication
#. Define Bookmarks (optional)
#. Define Port Forwarding (optional)
#. Define Group Policy
#. Define Connection Profile

.. note:: The above steps can be completed manually as documented below or via the SSL VPN Wizard

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

.. rubric:: Enable WebVPN on the appropriate interfaces

::
  webvpn enable <ifname>

.. rubric:: Setup Local Users (Optional)

.. rubric:: Setup external authentication server (Optional)


.. rubric:: Define Bookmarks (Optional)

.. rubric:: Define Port Forwarding List (Optional)

::

  webvpn
    port-forward <pf-list-name> <local-port> <remote-ip> <remote-port>

.. rubric:: Define Group Policies

::

  group-polocy <name> internal
  group-policy <name> attributes
    vpn-tunnel-protocol ssl-clientless
    !
    webvpn
      port-forward name <pf-list-name>
      port-forward enable <pf-list-name>

.. rubric:: Define Connection Profile

::

  tunnel-group <name> type webvpn
  tunnel-group <name> general-attributes
    default-group-policy <name>
  tunnel-group <name> webvpn-attributes
    group-alias <name>
