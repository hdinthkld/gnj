####################################
Cisco Dynamic Multipoint VPN (DMVPN)
####################################

.. toctree::
  :titlesonly:
  :maxdepth: 2

  DMVPN_BASIC_CONFIG


Introduction
============

Dynmaic Multipoint VPN is a solution developed by Cisco that aimed to make
large scale VPN deployments easier without needing to know all the endpoint
in advance.

It can be used for a traditional hub-and-spoke model especially when combined
with endpoints that do not have fixed IP addresses but it's really value comes
from allowing direct spoke-to-spoke traffic, removing the burdan and potentional
bottleneck from a central hub.

As well as the existing requirements for IPSec, three additional functional
components are needed:

* Multipoint GRE Interfaces
* Next Hop Resolution Protocol (NHRP)
* IPSec Proxy Instantiation

Multipoint GRE interfaces
=========================

One of the problems with having to establish a large number of individual GRE
tunnels is that each one takes up resources on the central router, the most
critical of which is the Interface Descriptor Block (IDB) for which there
are only a limited number.

In addition for traditional static GRE tunnels the source and destination
addressing needs to be known in advance which is not possible where VPN
endpoints could be using dynamic IP addressing.

Multipoint GRE, or mGRE solves both these issues.

An mGRE interface is a single interface from which all GRE tunnels from the
spokes can be termined.  This removes a lot of repetative configuration and
reducing the memory resource allocation as the mGRE interface only consumes a
single IDB regardless of the number of tunnels.

When configuring the mGRE interface, only the tunnel source is specified, not
the destination. An additional method is needed to communicate this information
so the hub is aware of the remote peers tunnel endpoint IP.  This is performed
by the Next Hop Resolution Protocol (NHRP)

Compared to individual tunnel interfaces, an multipoint interface is configured
with a single subnet not a subnet per tunnel.  This does cause issues with how
route selection is done with certain routing protocols.

Next Hop Resolution Protocol (NHRP)
===================================

NHRP can be thought of as a Layer 3 ARP.  Its role is to map the private IP
used in the DMVPN "cloud" against the remote peers current public IP.

The private IP will be used within all routing updates as the advertising
endpoint but it is the public IP that is needed in order to establish the
dynamic tunnel between two spoke sites.

The Hub will be configured to dynamically resolve NHRP requests based on its
available information.  At the start it will not know of any endpoints and
simply waits for a spoke to register and then for other spokes to query it for
information.

Upon initial bootup the spokes will establish a permenant tunnel with the spoke
and register their private IP against their public IP using NHRP. The first time
a spoke needs to communicate with a subnet located at a different spoke it will
ask the hub (via an NHRP query) for the public IP associated with the private IP
shown in the routing table.  One obtained the spoke will store this information
locally for a certain period of time and also update its CEF adjacency table to
allow for faster forwarding of packets.

Once the spoke knows how to communicate with another spoke it can dynamically
establish the VPN tunnel with the other spoke without any prior knowledge that
the spoke existed.

Dynamic IPSec Proxy Instantiation
=================================

In traditional VPNs it necessary to do one of the following

#. Manually configure an encryption domain ACL to specify what traffic should
   be permitted over the VPN.
#. Establish a GRE tunnel over IPSEC with the encryption domain being predefined
   as all GRE traffic between the two GRE peers.

If no prior knowledge of the routes or peers is known, neither option is
feasible.

Through NHRP a spoke is able to determine the remote spokes public IP which
can then be used as the IKE remote identity. Because the traffic is encapsulated
within GRE prior to being encrypted the subnets used over the VPN are not
relevant and the IPSec proxy profile will be the GRE traffic between the two
internal IP addresses just like with a traditional GRE-over-IPSec tunnel.

There is still however the issue of how to authenticate the spokes. A number
of methods can be used:

#. Pre-configure a unique key per IKE identity (spoke)
#. Pre-configure a global (wildcard) pre-shared-key for every IKE identity
#. Implement a public key infrastructure.

If any of the peers have dynamic IP addresses using a unique key is simply not
possible. Even if the peers are all static, having to manage all the keys
becomes increasing difficult as the number increases as all possible peers
need to be updated.

Using a global pre-shared-key would seem the simplest solution and in general
is widely used, it needs to be understood however that making updates to this
key is difficult and leads to a "flag day" where all devices need to be updated
at once. This is generally frowned upon by businesses and operations teams.

The final solution is to use a PKI, this is certainly the most scaleable
solution as it doesn't require any prior peer relationship configuration and
as long as no changes are made to the central CA, changes can be made (e.g.
reissuing of certificates) without affecting all devices.  It does however
require more prior work in order to implement the PKI and issue certificates
out to all participating devices.

Once a spoke knows what public IP to communicate with and how to authenticate
to it, the dynamic tunnel can be established and if successful traffic will
then flow directly between the spokes and not through the hub.

Establishing a Dynamic Multipoint VPN
=====================================

Below is a step-by-step review of how to setup spoke-to-spoke communication
within a DMVPN.

.. rubric:: Step 1: Setup the Hub site

Before any of the spokes can communicate it's necesary for the hub to be
established.  Because all the spokes will be configured with a static NHRP
entry for the hub, it cannot have any dynamic IP address.

On the hub, a point-to-multipoint tunnel will be created that has a static IP
address within the DMVPN Cloud,, this is known as the NBMA address and will
show up in various command outputs.

In order to secure the DMVPN traffic, the usual configuration elements required
for IPSec IKEv1 are required, namely:

#. IKE Phase 1 Policy
#. IPSec transform set
#. Authentication details (unique/wildcard PSK or RSA Signatures)
#. IPSec Profile

The tunnel interface will then be configured for protection using the IPSec
profile.

Note that a static encryption domain is not created.  These will be created
dynamically as required by each of the spoke-to-spoke tunnels.

Finally it is necessary for the hub to accept dynamic NHRP registrations and
queries. NHRP must be setup with a unique network ID for the DMVPN cloud as well
as an optional authentication password.  Various timers can also be adjusted
to determine how long to hold onto adjancy information given by the spokes.

Depending on the routing protocol used it may also be necesary to specify
certain additional properties.  This will be covered later.

.. rubric:: Steps 2:

Once the hub is ready to accept connections we can now configure the spokes.
Practically all of this configuration will be repeated for the spokes, the key
changes being:

* IP address assigned to the tunnel interface
* The networks that are advised through the tunnel as part of the routing
  protocol.

  As with any VPN tunnel, we first need to configure the IKE Phase 1,
  authentication details and IPSec Phase 2 Transform set.  These details
  should match what has been configured on the hub.

  Next the mGRE tunnel interface needs to be configured.  This is where
  the bulk of the configuration will be placed as it is necessary to:

  #. Define the source IP/interface of the tunnel
  #. Define the tunnel (private) IP address of the spoke
  #. Define the private and public IP address of the Hub
  #. Define which IP address to query for NHRP mappings, the next hop server
  #. Define the network ID for this DMVPN network
  #. Configure any properties specific to the routing protocol
  #. Specify which IPSec profile to use to encrypt the traffic

  It is not necessary to define the actual Hub as the tunnel endpoint as this
  will be handled by the Next Hop Server.  This tunnel interface will be used
  for all connections, both hub and spoke.

DMVPN Redundancy
================

DMVPN redundacy can be implemented using one of two methods:
#. Configure redundant hubs
#. Configure seperate mGRE subnets, effectively to seperate DMVPN networks

With a single DMVPN and redundant hubs the network is subject to the limitations
of a single routing protocol.  Where as if two DMVPN subnets are used it allows
for flexibility in the assignemnt of unique routing protocols and cost metrics
for the two networks.

In IOS versions prior to 12.(4.2) it was necessary to assign a unique IKE
identity (IP address) per DMVPN.

In the case of a single DMVPN network with dual hubs the decision on which hub
to use is left to the routing protocol.  The secondary hub as well as acting as
an NHS will also be a client to the primary NHS. If the primary hub were to fail
eventually the records would time out and the spokes would query the secondary
NHS instead.  Routing between the spokes will be determined by the routing
protocol.
