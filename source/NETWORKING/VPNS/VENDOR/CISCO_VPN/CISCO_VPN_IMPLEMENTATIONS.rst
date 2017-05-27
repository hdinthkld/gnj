#########################
Cisco VPN Implementations
#########################

.. toctree::
  :titlesonly:
  :maxdepth: 1
  :glob:

  IKEv1 <CISCO_IKEV1/IKEV1_OVERVIEW>
  IKEv2 <CISCO_IKEV2/IKEV2_OVERVIEW>
  AnyConnect <ANYCONNECT/ANYCONNECT_OVERVIEW>
  Dynamic Multipoint VPN <DMVPN/DMVPN_OVERVIEW>
  FlexVPN <FLEXVPN/FLEXVPN_OVERVIEW>
  GetVPN <GETVPN/GETVPN_OVERVIEW>
  CISCO_IKEV1/S2SVPNASA
  CISCO_IKEV2/S2SVPNIOS
  IPSEC/IPSEC_CONFIG_EXAMPLES
  GETVPN/GETVPN_FUNDAMENTALS
  GETVPN/GETVPN_KS_CONFIG
  GETVPN/GETVPN_GM_CONFIG


VPN Support for IOS and ASA devices
===========================================

The table below lists which types of VPNs are supported on each major device type:

============= === ===
VPN           IOS ASA
============= === ===
Site-To-Site   Y   Y
Remote Access  Y   Y
SSL            Y   Y
DMVPN          Y   N
GETVPN         Y   N
FlexVPN        Y   N
============= === ===

Functional Components
=====================

.. rubric:: ISAKMP Policy

Defines the acceptable algorithms, authentication types and lifetimes for the
Phase 1 Security Association.

This policy is defined globally as the correct selection has to be done before
anything else as part of the Main Mode exchange

.. rubric:: Crypto Keyrings

When pre-shared key authentication is being used the device needs to know what
is the valid authentication key sent by it's peer.  Whilst these can be defined
globally a crypto keyring makes them more manageable and also supports multiple
different network VPNs when defined as VRFs.

If a VRF is specified the keyring is associated the front-door VRF (FVRF)
specified in the configuration. The FVRF is the VRF which receives the encrypted
packet, compared with the inside VRF (IVRF) which is the internal/private
network for post-decryption forwarding lookups.

.. rubric:: ISAKMP Profiles

An ISAKMP Profiles is seen as a placeholder for putting Phase 1 and Phase 1.5
(XAUTH/Mode Config) configuration that applies to multiple peers.

An appropriate ISAKMP Profile is choosen by the device from either:

* The Phase 1 IKE Identity such as IP address, FQDN, USER-FQDN, DN or ID_KEY_ID
  when pre-shared keys are being used
* Fields matched from the certificate forwarded by the peer during
  authentication if RSA signature authentication is used

In order for fields to be matched form the certificate a mapping configuration
must also be configured and specified in the ISAKMP Profile.

.. rubric:: IPSec Proposal

The IPSec Proposal, more commonly called the Transform Set specifies the
encryption and integrity algorithms to use for the Phase 2 Security Association.

One other property that is often set is whether to use transport or tunnel mode.
On most Cisco devices the suggestion is to use Tunnel mode so this won't appear
by default in most configuration outputs.

.. rubric:: IPSec Profile

The IPSEC Profile is used to combine the IPSec Proposal along with the
IKE (either v1 or v2) profile.

In addition Perfect Forward Secrecy and lifetime settings can be changed if
needed.

Once defined this profile is then bound to the Tunnel Protection Profile so that
peers are checked for the correct credentials prior to being allowed access to
the network.

.. rubric:: Encryption Domain

The encryption domain specifies what traffic should be sent through the VPN
tunnel. This is communicates as part of the Phase 2 exchange and any mismatch
can lead to either the VPN only working intermittantly or not working at all.

For legacy VPN configuration using crypto maps (such as on the Cisco ASA
firewalls) this is defined using an ACL with permit statements specifying what
should be encrypted and denies being traffic that should be sent in plain text.

In many of the newer IOS VPN implementations the Encryption Domain is defined
automatically and can still be seen using the older commands.

When using GRE over IPSEC for instance the encryption domain is specified as all
GRE traffic (IP Protocol 47) from local peer ip (tunnel source) to remote peer
ip (tunnel destination)

However when using Raw IP within IPSec, the encryption domain is specified as
all IP traffic regardless of source/destination.

In both cases traffic destined for a configured tunnel interface will define
what traffic is encrypted or not.


.. rubric:: Crypto Maps

Crypto Maps are still the primary means of linking all the various components
in a VPN together on a Cisco ASA Firewall. The Crypto map will define the peer,
IPSec Proposal/Transform set, IPSec SA lifetime and encryption domain for that
specific VPN tunnel.

Crypto maps are defined globally and then linked to the interface from which VPN
connections should be established.

Whilst IOS VPNs can still use a crypto map and is still commonly done when
connecting to other parties. It is more common to use the method discussed
below.

.. rubric:: Tunnel Protection Profiles

To be able to more closely link the requirements of a VPN to it's correct
interface Cisco implemented the Tunnel Protection Profiles.

Unlike Crypto Maps, Protection Profiles are specified on the correct tunnel
interface and therefore allow encryption to be added after the fact to other
tunnel encapsulations.  The mode of tunnelling set on the interface determines
how the tunnel is implemented and it's important (as with any VPN) that both
peers are configured in a similar (if not exactly the same) manner. Both a
IPSec Profile and ISAKMP Profile (part of the IPSec profile on older IOS
versions) can be specified allowing for a greater level of flexibility in what
clients should be allowed to use that specific VPN.

The Tunnel Protection Profiles is used almost universally across VTI, DVTI,
DMVPN and FlexVPN implementations.

.. rubric:: Connection Profiles

The Cisco ASA offers a consistent means of defining VPN settings across all
the supported VPN protocols (whether it be IPSEC, SSL, or L2TP/IPSEC).  The
connection profile (or more usually called Tunnel Group from the CLI command)
defines the VPNs that are permitted.

In the case of traditional site-to-site VPNs the Connection Profile will be
named after the remote peers IP.  In the case of Remote Access VPNs that name
will be the "Group Name" for the legacy IPSec VPN Client.

For SSL/IKEv2 based VPNs the Connection Profile will be appended to the servers
name and provides a means to identify what service the user is trying to access.

Irrespective of what profile the user tries to connect to they can be moved to
a more appropriate group during the authorisation phase.
