########################
Virtual Private Networks
########################

.. toctree::
  :titlesonly:
  :maxdepth: 1

  IPSEC/IPSEC_OVERVIEW
  SSL/SSL_OVERVIEW
  VENDOR/CISCO_VPN/CISCO_VPN_IMPLEMENTATIONS

Introduction
============

* Why use VPN

VPN Protocols
=============

Over the years a number of different VPN protocols have come into existance.
Whilst some no longer see wide spread use it's possible you may come across
them when migrating from legacy technologies.

Broadly speaking the categories of VPN can be broken into:

* IPSEC (IKEv1, IKEv2)
* SSL (SSTP, OpenVPN, AnyConnect, GlobalConnect)
* Layer 2 VPNs (L2TP, GRE, MPLS)
* Legacy (PPTP)

.. _ref-ipsec:

IPSEC
-----

Can operate in the following modes:
 * IKEv1 / IKEv2
 * Site-To-Site or Lan-To-Lan
 * Remote Access
 * DMVPN
 * GetVPN (GDOI)

.. _ref-ssltls:

SSL/TLS
-------

Can operate in the following modes:
 * Clientless
 * Thin Client
 * Thick/Fat Client (TLS/DTLS)


.. _ref-gdoi:

GDOI
-----

 * Used with MPLS networks to offer confidentiality and integrity services whilst still maintaining the any-to-any feature of MPLS
 * GDOI is used in GetVPN alongside IPSec to provide the key distribution services between peer in the same group.

.. _ref-l2tp:

L2TP
-----

For further details on :term:`L2TP`, see:

.. _ref-pptp:

PPTP
-----

For further details on the :term:`PPTP`, see the :ref:`PPTP External Reference <ext-ref-pptp>`


.. _ref-sstp:

SSTP
-----

For further details on the :term:`SSTP`, see:
