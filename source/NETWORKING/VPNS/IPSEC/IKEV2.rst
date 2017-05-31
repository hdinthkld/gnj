##############
IKE Version 2
##############

Overview
============

IKE Version 2 or simple IKEv2 for short was published in RFC4306 but has since
been updated by a number of other RFCs, the latest being RFC 7296.  It offers
a number of improvements over the previous incarnation where weaknesses have
been identified. Ihis is mostly a benefit it must be understand that
the two versions whilst still similar are incompatible with each other and
therefore both peers must support the new version of the protocol.

The establishment of an IKEv2 VPN starts with an IKE_SA_INIT meesage whereby
both peers negotiate a set of algorithms and agree on the secret key that is
derived from additional key material. Additiona random data is added in the form
of a nonce. Once established all furter messages will be encrypted and the
integrity validated.

The second part of the exchange called an IKE_AUTH is secured using the agreed
properties from the first exchange.  Each side of the exchange will authenticate
each other using an integrity check generated from the secrets associated
with their identity.

Peers will then exchange algorithms that will be used to protect one or more
IPSEC SAs using either ESP or AH that is then used to secure the actual user
traffic.

Each time an additional IPSec is required (Such as adding a new subnet to be
encypted), an additional IKEv2 exchange will occur called a CREATE_CHILD_SA.
In the case of IKEv22 these exchange are not referred to as Phase 2 SAs but
instead called a Child SA.

Nonce
-----

In order to make the output of the cryptographic function less predicatable.
A certain amount of randomness is added.  This is the purpose of the nonce and
is a randomly generated value that is used as input to the data when
authenticating peers.

The nonce is sent by the initiator.

The nonce must be a minimum of 128 bits an up to 2048 bits in size.  In practice
the nonce must be at least half the size of the output for the PRF function that
is negotiated.

Denial of Service Protection
----------------------------

In order to offer more protection against Denial of Service IKEv2 provides
defences in the form of cookies. These cookies are designed to defend against
blind spoofing attacks as the initating party must be able to reply to the
cookie request which is a lot less computationally expensive that a real
IKEv2 exchange.

In the event an IP address is spoofed, no reply will be forthcoming that the
receiving party will not have wasted a lot resource establishing and storing
the intial state for a peer that doesnt exist. The point at which this
protection is enabled is configurable when a certain number of half connections
are seen.

This Denial of service protection only helps against attacks that are using
faked IP addressing, in the situation where the real host (such as one involved
in a botnet) is making the request and is able to reply resources would still
be consumed.  In the case of certain vendors, this protection is not enabled
by default (such as Cisco).

When enabled the initial IKE_SA_INIT is increased from a two way exchange to a
four way exchange as the cookie needs to be send and received before the
IKE responder will sent cryptographic material.

Certificate Request
-------------------

In IKEv1 when certificate authentication was used the subject name of the CA was
included.  This introducted the risk that it could be duplicated.

In IKEv2 this has been removed and now the responder can request that the
initiator use a certifiate issued from a preferred cerificate authority by
sending the SHA-1 hash of the public key of any trusted CA. The peer receiving
the request can select a CA from one of those in the request.

Out-of-Band Certificate Lookup
------------------------------

Instead of receiving the request directly in the IKE exchange a peer can notify
of it's ability to retrieve a certificate via HTTP instead.

The sending peer will then provide the 160-bit SHA-1 hash of the certifiate
along with the URL where it can be be downloaded from.  This is useful in
situations where fragmentation may occur due to the size of the certificates or
certificate chain.

It should be noted that this method of authentication is not widely supported
by all devices/vendors.

Extensible Authentication Protocol (EAP)
----------------------------------------

IKEv2 includes support for EAP as a valid method of authenticating an initiator.

The responder must use a form of certificate (either RSA or ECDA) as it's
means of authentication when a reply is sent to the initiator. In this instance
the initiator will repond first with it's identity.

Initial Contact
---------------

In the event of a VPN peer crashing or other issue that causes it to loose
the current cryptographic material IKEv2 includes a mechanism in which to
detect if an existing SA is in place and to tear it down so a new session can
be established.

This is based on the premise that there is no point waiting resources trying to
send encrypted packets to an endpoint that has no means to decrypted them.
It is better to simply tear it down and start from scratch by agreeing new
keying material.

Dead Peer Detection
-------------------

IKEv1 included a means of sending keepalives to ensure that its peer were
active and should be able to receive data. This however was wasteful on
resources especially on a busy hub.

An improvement was added to IKEv1 was an option called Dead Peer Detection.
Whilst similar to keepalives, it is far more scaleable as it only sent
keepalives when no data was being seen from that remote peer. As long as data
was being seen or no data needed to be sent, the DPD mechanism would do nothing.

One issue does exist however, because DPD will only sent a request when traffic
needs to be sent over the VPN it may not notice the failed peer straight away.
By using routing protocols over the tunnel however, this should not be an issue
as they will notice the failure more quickly and where redundant links exist
will reconverge as necessary.

It is recommended that DPD be enabled so that in the event of failed peers the
now unnecessary security assoications can be torn down. On the hub however if a
large number of VPN peers fail this may place a lot of additional burden on this
central device.  Whilst DPD should be enabled on all spoke devices, care should
be taken not to set the DPD timeout values too aggressive so that it doesn't
support from a Denial Of Service simply due to having to process to many DPD
request/replies.


NAT-T
-----

NAT-T offers a mechanism by which IPSec traffic can be sent through a NAT
device. It was an optional features in IKEv1 but is mandatory in IKEv2.

NAT-T works by encapsulating the ESP packets within a UDP packet (usually using
UDP 4500) so that the NAT device can individually track the connections coming
through it based on the source/destination port numbers.  Prior to NAT-T it was
necessary for devies to support IPSec Passthrough whereby the needed to keep
track of the SPI information in the ESP headers to know what to do and this
ofter limited the ability of the feature (for example only one IPSec device
at a time).

One problem that exists is when an IPSec connection is idle it isn't sending
or receiving any data so this would cause the NAT gateway to start the idle
timer because according to RFC 3715 NAT entries should have finite lifetime.
This is taken care of by sending NAT-T keepalives at certain intervals to avoid
the NAT gateway tearing down the sessions.  It should be noted that these
keepalive packets whilst not carrying any protected data, are not encrypted
in any way. A single payload of 0xFF is included in the packet.

Note that because of the way in which Authentication Header (AH) validate the
packet, it is still not possible to use AH and NAT with IKEv2 just as with
IKEv1.

NAT detection is included as part of the protocol and when detected the peers
will automatically switch to encapsulating the IKE and IPSec sessions within
UDP (IP Protocl 17) over port 4500 (unless configured otherwise).

IKEv2 and IKEv1 Comparison
==========================

As previously mentioned although they perform the same function IKEv1 and IKEv2
and not compatible.

Two of the biggest advantages of IKEv2 are its inclusion of Next Generation
Encryption (NGE) and anti denial-of-service capabilities.

IKEv2 also includes EAP authenticaton which was not available as part of IKEv1.

The overall packet structure of IKEv2 has also been redesigned to be more
efficient, needing fewer packets and less bandwidth that IKEv1.

The current form of IKEv1  is the combination of numerous RFCs which have
resulted in a number of bolt-on features being added to IKEv1.  Many of these
features are now included in IKEv2 by default and therefore allowed for a
cleaner design.

IKEv2 no longer provides for Main Mode or Aggressive mode, instead there is a
single IKE_SA_INIT as covered above.

In IKEv1 the lifetime of the IKE SA was negotiated in the first pair of
messages. This resulted in incompatibilities due to mismatched values. For IKEv2
the lifetime is now a locally configured value which is not negotiated between
peers.  A peer is responsible for deleting or rekying the SA when it's local
lifetime expires.

Whilst ECDSA authentication was introduced to IKEv1 it was late coming and
therefore has seen little adoption.  It has however seen wide adoption in IKEv2
and a requirement for implementations to support the Suite B Profile (RFC 6380)

IKEv1 has no support for high availability resulting in tricks such as DNS
round robin or using a dedicated load balancer.  IKEv2 provides for a
redirection ability where a client can be told to connect to another VPN
gateway.

The IKEv2 provides for a wider range of traffic selectors as well as allowing
the responder to select a narrower set than the initiator proposed, this can
be done per Child SA.  The official standard allows for a single IPSec SA to
handle both IPv4 and IPv6 traffic however not allow Vendor (including Cisco)
provide support for this.

NAT has already been covered above, NAT-T and NAT detection were originally
not included as part of the IKEv1 standard but they are now as part of IKEv2.

IKEv1 did not include any method for pushing configuration to peers (such as
IP addressing in the case of a remote access client).  IKEv2 has inbuilt
support for configuation data exchange via the use of a configuration payload.

In IKEv1 when pre-shared keys were used it was not possible to based identity
on anything other than the peer IP address. Because IKEv2 does not use the
shared secret as part of its initial calcuation other identity details can
be used. It is assumed that because the peer was able to perform these
calculations that it must have possesion of the private key and therefore its
identify can be more trustworthy.

Because of the above IKEv1 needed to have a unique IP per pre-shared-key. IKEv2
does away with this limitation.

IKEv2 is also considered more reliable as message exchanges are sent in pairs
so a response is always expected.

In IKEv1 all sets of cryptographic ciphers are transferred in seperate
transforms. In IKEv2 all algorithms are sent within a single transform, or two
where combined and non-combinded mode ciphers are used.

IKEv2 is able to provide combined mode ciphers in which a single algorithm
is able to perform both encryption and integrity protection.

In IKEv1 it was possible for an IPSec SA to exist without a corresponding IKE
SA. This is due to IKEv1 not operating in a noncontinuous channel mode.  In
IKEv2 this is not possible as for an IPSec SA to exist it must have a
matching IKEv2 SA.
