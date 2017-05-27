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

.. rubric:: What is a VPN

VPNs are a private network service delivered over a public (or shared) network
infrastructure.  The two important characteristics of a VPN are that it is
virtual and that it is private.

The simplest form of a VPN connection could be considered a telephone call
because it is virtual in the sense there is no direct physical
link between the two parties and that it is private therefore protected (at
least at a simple level) from eavesdropping.

.. rubric:: Motivations for VPN usage

The primary motivation for deploying a VPN is in order to save cost. Whilst
there is no real technical limitation in providing all of an enterprises
locations (be they head office, branch office or remote worker) with a dedicated
connection, the cost to do so would be unrealistic for most businesses.

Practically all enterprises or organisations need some kind of Internet access
therefore why not utilise this existing connectivity to connect all of the
distant offices.

Add to this this the requirement for users to be able to acces company resources
whereever they may be (perhaps also using wifi, mobile or even satellite
technology), the benefits of providing VPN services start to outway the
downsides.

The majority of organisations won't use VPN's exclusively and will make use
of local WAN services (such as MPLS) where it is feasable. VPNs will then be
used to provide connectivity to those offices where direct WAN connectivity is
not possible (such as road warriors and/or mobile sites).  This sites will make
use of whatever connectivity they have available to connect back to a central
site and then onto the larger WAN.

.. rubric:: Risks of using a VPN

The cost benefits of offering VPN services even with the need for specialised
devices to terminate the connections soons becomes clear, it's not without it's
downsides.

The most obvious limitation is that there should be no assumed security of data
when it is carried over the public network. Additional techniques are needed to
secure the data which then carries it's own problems in terms of complexity and
impact on performance.

The second risk is that public networks don't offer any performance guarantees.
Usually when choosing your private WAN supplier, part of the contract with them
will include service level agreements (SLAs) that dictate what is acceptable
performance (e.g. packet loss, round trip time, jitter, etc) and the supplier
may be forced to pay penalties if this is not met, often Quality-Of-Service
technology will be used to ensure these targets are met. None of this applies
when running a VPN over a public network (especially the Internet).


VPN Technologies
================

Over the years a number of different VPN protocols have come into existance.
Whilst some no longer see wide spread use it's possible you may come across
them when migrating from legacy technologies.

Broadly speaking the VPN technologies can be broken into:

* Layer 2 VPNs
* Layer 3 VPNs
* SSL VPNs

The way in which a VPN is used also has an impact on the VPN protocols
development. The VPN technologies can again be categories as used for either:

* Site-To-Site
* Remote Access

.. rubric:: Layer 2 VPNs

Layer 2 VPNs as their name suggests operate at the data-link layer of the OSI
model. This means that they are independant of the protocol used to transfer
the users data which would commonly operate at Layer 3 (such as IPv4, IPv6, etc).

The most common Layer 2 VPNs are ATM and Frame Relay.  Because of the way in
which the protocols operate within a service providers infrastructure, they are
inherintly only designed for site-to-site connectivity.

.. rubric:: Layer 3 VPNs


Layer 3 VPNs operate at the network layer of the OSI model and therefore are
tied to the protocol operating at that layer (such as IPv4, IPv6, etc). This
could be considered a limitaion but it enables traffic to be carried over any
network that supports the same Layer 3 protocol (such as the Internet).

The most commonly deployed Layer 3 VPNS are IPSec, GRE and MPLS.

*IPSec* is an industry standard VPN Solution that has been added as an extention
to IPv4 but is built-in to the protocol for IPv6. In reality it is is not a
single protocol and requires assistance from other protocols such as *ISAKMP* and
*Diffie-Helman* to function. The most commonly deployed form of IPSec using *IKE*
version 1 but with the increasing need for better security a newer version of
protocol exists which not only helps standardise the original but also takes
away some of it's weaknesses.  Probalby unsuprisingly this new version is called
IKEv2.

Because of IPSecs functionality it can operate as either a site-to-site or
remote access VPN solution. This flexibility does however come at a cost as
whilst site-to-site connectivity is mostly standardised, anything beyond the
most basic remote access solution ofter requires buying into a particular
vendors implementation. IPSec clients are built-in to most modern operating
systems that provide this basic connectivity either alone or when combined with
*L2TP*.

The *L2TP* protocol (RFC 2661) was designed for transporting PPP frames over an
IP network. This was in the past used to allow users to dial-in to a local
access server and then be able to transparantly access the corporate network
over the service provider network via the L2TP tunnel.

L2TP however does not offer any confidentiality guarantees so anyone with access
to the service provider equipment could still spy on private data.  When
combined with IPSec however this limitation goes away and is a type of remote
access service built-in to most of the popular operating systems (such as
Windows, Android, etc). Because of the IPSec encapsulation it is not necessary
to expose the L2TP services to the wider Internet, only those required for the
IPSec (e.g. ISAKMP, ESP/AH and NAT-T) communication need to be available.

*GRE* VPNs were originally developed by Cisco but then later published as a
standard under RFC 1701 with the delivery header being defined in RFC 1702.
Although GRE is considered a VPN because the data is encapsulated with the GRE
headers, GRE as with L2TP does not offer any confidentiality guarantees. Even
with limitation it is still very useful as when combined with IPSec it removes
IPSECs limitation of only being able to carry IP traffic whilst also benefitting
from the confidentiality features provded by IPSec.

GRE in general should be considered a site-to-site VPN technology and not
suitable for remote access purposes, however with various Cisco designs it is
certainly possible to deploy GRE as a rapid site deployment solution.  Just
don't expect this to be something your average road warrior would be want to
deal with.

*PPTP* was originally developer a number of vendors (including Microsoft) and
whilst a specification was published under RFC 2637 it has not been proposed as
an IETF standard.  It was originally included with Windows 95 OSR2 in 1999.

PPTP works by initiating a connection over TCP 1723 and the carrying the user
traffic over a non-standard GRE packet format. Over the years a number of
weaknesses have been found in the protocol therefore it is not recommended for
any new deployment and should only be used as a last resort with non-critical
data.

*MPLS* is a type of VPN that is most commonly deployed across a service
providers network. By using multiple levels of MPLS labels (or tags) its
possible for a service provider to use the same network infrastructure to
support multiple customers whilst also keeping all of the IP traffic
(or other layer 3 protocols) and addressing seperate from one another.
As a side note, MPLS technically operates between Layer 2 and 3 due to the way
it encapsulates packets.

MPLS by default does not offer any confidentiality guarantees so requires
support from other VPN technologies to provide this.  As MPLS is not deployed
over a public network (such as the Internet) the majority of customers are not
as concerned as with other environments. Those enterprises that do have strict
compliance requirements over data privacy can choose to implement VPNs (such as
IPSec) over the top of MPLS but this breaks the inherent benefit of the any-to-
any connectivity that MPLS provides.

Cisco has developed a type of VPN that overlays MPLS whilst still maintaining
the any-to-any connectivity and also making the VPN infrastructure more
managable by having the majority of the VPN configuration stored in a central
location and pushed out to the network endpoints that are members of a group.
This technology is called *GetVPN* and combines IPSec with another protocol,
*GDOI*.

.. rubric:: SSL VPNs

Most recently there has been a take up on a type of VPN that takes advantage of
existing technologies to secure user traffic.  This type of VPN is known as an
SSL VPN and enables users to access the internal network securely over an
untrusted network by using the the same (or similar protocols) to that used for
normal Web Traffic.

SSL VPNs would be using the same SSL (or newer TLS) protocols to offer the
confidentiality and integrity guranatees expecting of a secure VPN.

Depending on the the deployment users can access the SSL VPN either directly
through a supported Web Browser and/or through a full VPN Client just as with
the above protocols.  The more appropriate method will of course depend on
what the user needs access and will be converted later in the relevent chapter.

A final note is that unlike with the earlier protocols, SSL VPNs are inherently
designed based on a particular vendors implementation so it should not be
expected that one vendor would be able to communicate with another vendors
device using this technology.

SSL VPNs go by various names, some of the most common are:

* Cisco AnyConnect
* Palo Alto GlobalConnect
* Microsoft SSTP
* Pulse Secure
* OpenVPN

Not all of these implementations should be considered equal, for instance whilst
OpenVPN and Microsoft SSTP are purely VPN clients the others offered by Cisco
and Palo Alto (as 2 examples) offer a more complete solution including
extra features suchas Endpoint protection.
