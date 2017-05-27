##########################################################
Cisco IOS IKEv1 VPN Legacy Crypto Map with Pre-shared Keys
##########################################################

In this section we will configure a pair of Cisco IOS routers to communicate
over IPSec using IKEv1 using the older crypto map style of config and
pre-shared key authentication

It is assumed that the router already has basic IP connectivity to the public
WAN and all private interfaces are configured. The default route is also assumed
to be via the public WAN.

#. Define the pre-shared key for the remote peer
#. Define the Phase 1 ISAKMP policy
#. Define the Phase 2 IPSec Proposal and set the VPN encapsulation method
#. Define the Encryption Domain for the traffic which should be sent over the
   VPN
#. Combine all the various settings into a crypto map
#. Apply the crypto map to the public WAN interface

Configuration Steps
===================

Step 1: Define the pre-shared keys
----------------------------------

.. code-block:: none

  crypto isakmp key <psk> address <ip-of-peer>

Step 2: Define the Phase 1 ISAKMP policy
----------------------------------------

.. code-block:: none

  crypto isakmp policy <priority-number>
    encryption <encryption-algorithm>
    hash <integrity-algorithm>
    group <dh-group>
    lifetime <seconds>
    authentication pre-share

Step 3: Define the Phase 2 IPSec Proposal
-----------------------------------------

.. code-block:: none

  crypto ipsec transform-set <ts-name> <encryption-algorithm> <hashing-algorithm>
    mode tunnel

Step 4: Define the Encryption Domain
------------------------------------

To define the traffic to be encrypted an ACL needs to be created.

Each entry in this access list will create a new Phase 2 Security Association
which will take up resources on the VPN gateways.  Where it is possible to do
so summarisation of networks should be done and also avoid the use of
per-host ACE's or those specifying individual ports and protocols (interface
ACLS should be used for that purpose)

.. code-block:: none

  access-list <acl-id-or-name> permit <local-net> <local-wildcard> <remote-net> <remote-wildcard>

Step 5: Define the crypto map
-----------------------------

.. code-block:: none

  crypto map <cm-name> <seq-number> ipsec-isakmp
    match address <acl-id-or-name>
    set transform-set <ts-name>
    set security-association lifetime seconds <seconds>
    set peer <ip-of-peer>

Step 6: Bind the Crypto Map to the receiving interface
------------------------------------------------------

.. code-block:: none

  interface <type><slot/num>
    crypto map <cm-name>


Complete example
================

The example below is based of the below topology:

.. todo:: Insert topology image

On the VPN Hub configure the following:

.. code-block:: none

  crypto isakmp key mysecretkey address 192.168.2.2

  crypto isakmp policy 10
    encryption aes
    hash sha
    lifetime 86400
    group 14
    authentication pre-share

  crypto ipsec transform-set ESP-AES128-SHA1 esp-aes 128 esp-sha-hmac
    mode tunnel

  ip access-list extended EACL-R1-TO-R2
    permit ip 10.1.0.0 0.0.255.255 10.2.0.0 0.0.255.255

  crypto map CM-PUBLIC-WAN 10 ipsec-isakmp
    match address EACL-R1-TO-R2
    set peer 192.168.2.2
    set transform-set ESP-AES128-SHA1
    set security-association lifetime seconds 28800

  interface FastEthernet0/0
    crypto map CM-PUBLIC-WAN

On the VPN Spoke configure the following:

.. code-block:: none

  crypto isakmp key mysecretkey address 192.168.1.1

  crypto isakmp policy 10
    encryption aes
    hash sha
    lifetime 86400
    group 14
    authentication pre-share

  crypto ipsec transform-set ESP-AES128-SHA1 esp-aes 128 esp-sha-hmac
    mode tunnel

  ip access-list extended EACL-R2-TO-R1
    permit ip 10.2.0.0 0.0.255.255 10.1.0.0 0.0.255.255

  crypto map CM-PUBLIC-WAN 10 ipsec-isakmp
    match address EACL-R2-TO-R1
    set peer 192.168.1.1
    set transform-set ESP-AES128-SHA1
    set security-association lifetime seconds 28800

  interface FastEthernet0/0
    crypto map CM-PUBLIC-WAN

Verification
============

Once the VPN configuration has been applied, it will likely be necesary to
generate some traffic which matches the encryption domain in order for the
vpn establishment to start.

In this example we have a loopback interface configured on both the hub (
10.1.1.1) and spoke (10.2.1.2) so initiating a ping between these hosts should
be sufficient.

Because the Hub could be busy with dealing with other VPNs, lets does this on
the spoke instead as follows:

.. code-block:: none

  ping 10.1.1.1 source 10.2.1.2

The first few pings are likely to fail whilst the VPN is coming up but after
that they should reply without issue.

If the pings are replying we can probably assume that the VPN is up but how
do we know for sure.

Firstly lets check if the Phase 1 SA is up:

.. code-block:: none

  show crypto isakmp sa detail

The output should be similar to that below:

.. code-block:: none

  C-id  Local           Remote          I-VRF  Status Encr Hash   Auth DH Lifetime Cap.
  1001  192.168.2.2     192.168.1.1            ACTIVE aes  sha    psk  14 23:59:53



If the status is showing a ACTIVE that is good as it means the VPN is believed
to be stable and no further action is being taken.  If is saying anything else
it could indicate the VPN is having issues or that it is renegotiating (such as
during a rekey after the lifetime has expired).

We can also see that the Phase 1 properties have negotiated to what we
configured.


Assuming all is well, lets check that packets are being successfully encrypted
and decrypted as follows:

.. code-block:: none

  show crypto ipsec sa peer 192.168.1.1

And the output should then be as follows:

.. code-block:: none
  :emphasize-lines: 9-10

  interface: FastEthernet0/0
    Crypto map tag: CM-PUBLIC-WAN, local addr 192.168.2.2

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (10.2.0.0/255.255.0.0/0/0)
   remote ident (addr/mask/prot/port): (10.1.0.0/255.255.0.0/0/0)
   current_peer 192.168.1.1 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 9, #pkts encrypt: 9, #pkts digest: 9
    #pkts decaps: 9, #pkts decrypt: 9, #pkts verify: 9
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 192.168.2.2, remote crypto endpt.: 192.168.1.1
     path mtu 1500, ip mtu 1500, ip mtu idb FastEthernet0/0
     current outbound spi: 0xA464B844(2758064196)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0xAA697053(2859036755)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 5, flow_id: 5, sibling_flags 80004040, crypto map: CM-PUBLIC-WAN
        sa timing: remaining key lifetime (k/sec): (4311956/28771)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound esp sas:
      spi: 0xA464B844(2758064196)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 6, flow_id: 6, sibling_flags 80004040, crypto map: CM-PUBLIC-WAN
        sa timing: remaining key lifetime (k/sec): (4311956/28771)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

The key information here, is whether packets are being encrypted and decrypted.
If they are all is well and no futher action should be necessary.

Other details that can be found out are whether the correct encryption and
hashing is in place, whether PFS is being used, if reply detection is enabled
and finally the remaining lifetime of the IPSec SA.

The SPI's shown for both the inbound and outbound direct can be useful when
performing packet captures as part of troubleshooting as they are not encrypted
so can be used to identify a given VPN connection where possible a few are
coming from the same IP address (e.g. with multiple ACE entries in the
encryption domain)

.. _ipsec-ios-legacy-ikev1-psk-troubleshooting:

Troubleshooting
===============
