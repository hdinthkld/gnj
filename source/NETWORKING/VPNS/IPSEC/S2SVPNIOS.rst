$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Fundamentals of IPSec Site-To-Site VPNS
$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$

Implementing Site-To-Site VPNs on IOS Routers
=============================================

Legacy IKEv1 using Pre-Shared Key
---------------------------------

Configuration
#############
* Configure Interfaces as appropriate
* Configure Routing as appropriate
* Configure Phase 1 (ISAKMP) Policy ::
  ! Number represents priority
  crypto isakmp policy 1
  authentication pre-shared
  encryption <encryption algorithm>
  hash <hashing algorithm>
  group <DH Group>
  lifetime <seconds>
  
* Configure Pre-shared keys ::
  crypto isakmp key <very-secret-key> address <peer-ip>
    
* Configure Phase 2 (IPSec) Transform Set ::
  crypto ipsec transform-set <ts-name> <encryption-algorithm> <hashing-algorithm>
     mode tunnel
     
* Configure Phase 2 (IPSec) Lifetime (Default if not defined in crypto map) ::
  crypto ipsec security-association lifetime seconds <seconds>
  
* Define traffic to be encrypted/unencrypted (Encrytion Domain or Crypto ACL) ::
  access-list [extended] <cacl-name-or-id> permit ip <local-subnet> <local-mask> <remote-subnet> <remote-mask>
  
* Put all the pieces together ::
  crypto map <cm-name> <seq-id> ipsec-isakmp
    match address <cacl-name-or-id> 
    set peer <peer-ip>
    set transform-set <ts-name>
    ! What about lifetime?
    
* Apply the configuraton to the appropriate interface ::
  interface <type> <slot/num>
    crypto map <cm-name>
    
Testing
#######
* Generate Interesting traffic that matches the configured encryption domain policy
* Confirm that response is received from remote peer using traffic encapsulated in the tunnel
* Verify that encryption/decryption counters are increasing


Verification
############
* Verify Interface status ::
  show ip int brief
* Verify Routing ::
  show ip route
* Verify bi-directional IPSec SA's exist ::
  show crypto engine connections active
* Verify that Phase 1 SA's are valid ::
  show crypto isakmp sa
* Verify that Phase 2 SA's are active and traffic is being encrypted / decrypted ::
  show crypto ipsec sa
  
Optional Components
###################
* Enable Dead Peer Detection (must be active both sides) ::
  crypto isakmp keepalive <interval> <retries> [periodic | on-demand]
  
  
Notes
#####
* Lowest lifetime will be used, they don't have to match ::
  ISAKMP:(0):Checking ISAKMP transform 1 against priority 10 policy
  ISAKMP:      encryption AES-CBC
  ISAKMP:      keylength of 256
  ISAKMP:      hash SHA
  ISAKMP:      default group 5
  ISAKMP:      auth pre-share
  ISAKMP:      life type in seconds
  ISAKMP:      life duration (basic) of 1200 <-- Sent from Initiator
  ISAKMP:(0):atts are acceptable. Next payload is 0
  ISAKMP:(0):Acceptable atts:actual life: 600
  ISAKMP:(0):Acceptable atts:life: 0
  ISAKMP:(0):Basic life_in_seconds:1200
  ISAKMP:(0):Returning Actual lifetime: 600 <-- Responder old initiator what to use as it has a shorter lifetime
  ISAKMP:(0)::Started lifetime timer: 600.
  
* When there is an issue with Phase 2, there will still be an active Phase 1 but no Phase 2 SA SPIs will be listed or IPSec connections will be listed

* When troubleshooting VPNs, necessary to enable debug as nothing will be logged regardless of success/failure. Be aware of resource utilisation if this is a multi-function/customer device ::
  debug crypto isakmp sa
  debug crypto ipsec sa
  logging buffered debug
  
* When using ESP, then mode is not taken into account as Tunnel is always used
* Encryption domains do not need to match as long as the receiving peers encryption domain encompasses the one being sent. This however can lead to issues with the VPN needing to be established the opposite direction as it will then fail. 

  Best to just have them match otherwise it could lead to one-way tunnel establishment.


Common Errors
#############
* Pre-Shared Key Mismatch
  Sending endpoint will not receive a reply as the receiving endpoint will not respond but will log the following error ::
    CRYPTO-4-IKMP_BAD_MESSAGE: IKE message from 192.168.15.1 failed its sanity check or is malformed

* Mis-matched Phase 1 Policies
  * Encryption ::
      ISAKMP:(0):Encryption algorithm offered does not match policy!
      ISAKMP:(0): phase 1 SA policy not acceptable! (local <ip> remote <ip>)
    
  * Incorrect Encryption Key Length (AES)  ::
      ISAKMP:(0):Proposed key length does not match policy
      ISAKMP:(0): phase 1 SA policy not acceptable! (local <ip> remote <ip>)

  * Integrity (Hashing) ::
      ISAKMP:(0):Hash algorithm offered does not match policy!z
      ISAKMP:(0): phase 1 SA policy not acceptable! (local <ip> remote <ip>)
  
  * Key Exchange Method (DH Group) ::
      ISAKMP:(0):Diffie-Hellman group offered does not match policy!
      ISAKMP:(0): phase 1 SA policy not acceptable! (local <ip> remote <ip>)

* Mismatched Phase 2 Policies
  * Encryption, Hashing or transport/tunnel mode ::
  IPSEC(ipsec_process_proposal): transform proposal not supported for identity: {esp-des esp-sha-hmac }
  ISAKMP:(1015): IPSec policy invalidated proposal with error 256
  ISAKMP:(1015): phase 2 SA policy not acceptable! (local <ip> remote <ip>)

* Mismatched Encryption Domains
  map_db_find_best did not find matching map
  IPSEC(ipsec_process_proposal): proxy identities not supported


GRE Over IPSEC
--------------

Configuration Steps
###################
# Setup Interfaces
# Setup Routing
# Create GRE Tunnel interface ::
  int tun 0
    ip address <ip> <mask>
    tunnel source <WAN Interface>
    tunnel destination <Peer IP>
    tunnel mode gre ip

    ! Optional put interface into routing process
    ip ospf 100 area 0

# Create ISAKMP Policy ::
  crypto isakmp policy 10
    encryption <algorithm>
    hash <algorithm>
    group <dh-group>
    authentication pre-share
    lifetime <seconds>
    
# Create Pre-Sharked Keys ::
  crypto isakmp key <key> address <peer-ip>
 
# Create IPSec Transform Set ::
  crypto ipsec transform-set <ts-name> <encryption-algorithm> <hashing-algorithm>
     mode tunnel

# Create IPEC Profile ::
   crypto ipsec profile <ipsec-profile-name>
    set transform-set <ts-name>
    
# Apply Profile to GRE Tunnel Interface
  int tunnel 0
    tunnel protection ipsec profile <ipsec-profile-name>
    
Optional Components
###################
* Restrict inbound traffic to VPN Protocols only, useful to enforce use of protection profile ::
  ip access-list extended IACL-WAN-IN
   permit ospf any any
   permit icmp any host 192.168.25.2 unreachable
   permit icmp any host 192.168.25.2 packet-too-big
   permit udp any host 192.168.25.2 eq 4500
   permit esp any host 192.168.25.2
   permit udp any host 192.168.25.2 eq 500
   deny ip any any log
  !
  interface <type> <slot/num>
    ip access-group <acl-name> in

Notes
#####
# Without the protection profile the tunnel will still work but data will not be encypted, use with cause
# When the protection profile is bound to the interface, a crypto map will be dynamically gnerated
# An IPSec Phase 2 SA will be created automatically (via dynamic acl)  which encapsulates all GRE traffic between the source and destination peers in ESP packets
# Standard troubleshooting tools can be used once configured
# Is not unusual to see more than 2 SA's per peer, this can occur when both peers try to initiate the connection at the same time.  The peers will determine for themselves which to use.
# In order for tunnel to be fromed on ESP, ISAKMP and UDP 4500 need to be allowed into the router

Common Errors
#############
* Mismatching Tunnel Protection featurs (configured one end, not the other) ::
  %CRYPTO-4-RECVD_PKT_NOT_IPSEC: Rec'd packet not an IPSEC packet. (ip) vrf/dest_addr= /<local ip>, src_addr= <remote ip>, prot= 47



Site-To-Site VPN with Self-Signed Certificates
----------------------------------------------

Configuration Steps
###################
# Set hostname ::

# Set domain name ::

# Generate signature ::
  crypto key generate rsa [modulus <bit-size>]

# Copy the key data displayed from the following command ::
  show crypto key mypubkey rsa 
  
# Import key into other router ::
  crypto key pubkey-chain rsa
    addressed-key <peer-ip>
    key-string
    ! Paste the key data from the previous step
    quit

# Copy the key from 2nd router and import into 1st router (or more as needed)

# Phase 1 Policy ::
  crypto isakmp policy 10
    authentication rsa-sig
    encryption aes
    hash sha
    lifetime 86400
    group 5

# Phase 2 Policy
  crypto ipsec transform-set <ts-name> <encryption> <hashing>
    mode tunnel
    
# Phase 2 Lifetime
  crypto ipsec security-association lifetime seconds <seconds>
  
# For Legacy VPN: Define Encrpytion Doamin and Crypto Map then bind to interface
# For Tunnel-based (e.g. GRE-Over-IPSEC): Define IPsec Profile and set it as the tunnel protection profile



Creating an IOS CA
------------------

Configuration
#############
# Ensure time on the CA server is correct (NTP is recommended)

# 
#crypto key generate rsa general-keys label IOS-CA modulus 2048 exportable

# Export Keys to flash
crypto key export rsa IOS-CA pem url nvram: 3des test1234

# Enable HTTP Server for SCEP
ip http server

crypto pki server <name>
  database level minimum
  database url nvram:
  issuer-name cn=<common> l=<location> c=<country>
  lifetime certificate <days>
  grant auto
  no shutdown
  
# Enter Private Key Passphrase
# CA Certificates will then be generates

Verification
############
show crypto pki server

Enroll an IOS device with the CA
--------------------------------

Configuration
#############
# Configure PKI Trustpoint
crypto pki trustpoint <tp-name>
  enrollment url <ca-scep-ip>

# Obtain CA certificates
crypto ca authenticate <tp-name>

# Enrol to the CA
crypto ca enrollment <tp-name>

Verification
############
show crypto pki certificate


Configure IOS Device to use the new certificate
-----------------------------------------------
# Ensure Phase 1 Policy is set to RSA signatures for authentication
# All other configuration steps are the same



Static VTI with Wildcard PSK
---------------------------------

Configuration
#############
# Phase 1 Policy
# Wilcard PSK (0.0.0.0/0.0.0.0)
# Transform-set
# IPSec Profile
# Tunnel Interface
  *  IP Address
  *  Source
  *  Destianation
  *  Mode - IPSec IPv4 (Direct IP into IPSec Encapsulation, no GRE Overhead)
  *  IPSec Protection Profile
  *  Include in Routing Protocol
# Repeat on other peer

Verification
############
show crypto engine connections active
show crypto ipsec sa
show crypto isakmp sa
show crypto map

Troubleshooting
###############
debug crypto isakmp
debug crypto ipsec


IOS Site-To-Site VPN with Aggressive Mode
-----------------------------------------

Configuration
#############
# Interfaces
# Routing
# Phase 1 Policy
# Configure ISAKMP Peer for Aggresive Mode ::
  crypto isakmp peer address <peer-ip>
    set aggressive-mode password <psk>
    set aggressive-mode client-endpoint ipv4-address <peer-ip>
# Transform set
# IPSec SA Lifetime
# Encryption Domain
# Crypto Map
# Bind Crypto Map to Interface

Verfication
###########
* Debug logs will show that Aggressive Mode exchange is being done

Troubleshooting


IOS Site-To-Site with Overlapping Subnet (Method 1)
---------------------------------------------------
Used when both sites have overlapping address space

This method, uses double NAT


Configuration
#############
# Interface Config
    Configure NAT inside/outside as appropriate
# Configure Policy NAT statement specifying the real source but the other parties NAT'd address range
# {hase 1
# PSK
# Transform set
# IPSEC SA Lifetime
# Crypto ACL
  * Specify NAT'd source and NAT'd destination
# Crypto Map
# Bind crypto map to interface

Notes
#####
* NAT'ing applies before it is determined if the traffic should be encrypted or not


IOS Site-To-Site with Overlapping Subnet (Method 2)
---------------------------------------------------
Used when both sites have the same IP address range at both sites but it's not possible to apply NAT to the remote peer

This method, uses source and destination NAT on a single router.

Configuration
#############
# Verify interfaces
# Verify routing
# Set NAT on both interfaces
# Set Source NAT from Local Subnet to Remote Subnet
# Set Destination NAT from Remote Subnet to Local Subnet
# Phase 1 Policy
# PSK
# IPSEC TS
# IPsec Lifetime
# Crypto ACL using Local's NAT subnet but remote's real subnet
# Crypto Map
# Bind Crypto Map to Interface
# Repeat on other peer

Verification
############
show crypto isakmp sa
show crypto ipsec sa

Troubleshooting
###############
debug ip icmp

Notes
#####
# This would only work when there is only a single remote site with an overlapping subnet, doesn't seem scalable to me?


IOS Site-To-Site with Keyring
---------------------------------------------------
Keyring provides a central means of storing PSK/RSA credentials. These Keyrings can then be bound to specific ISAKMP Policy

Configuration
#############
* ISAKMP Policy
* Define Keyring ::
  crypto keyring <keyring-label>
     pre-shared-key address <ip> key <psk>
* ISAKMP PRofile ::
  crypto isakmp profile <name>
    match identity address <ip>
    keyring <keyring-label>
* Phase 2 Transform Set
* IPSec SA Lifetime
* Cryto ACL
* Crypto Map
* Set Crypto Map to use ISAKMP Profile
* Bind to Interface

IOS Site-To-Site with Keyring and Self-Signed Signature
-------------------------------------------------------
Uses both a pre-shared key and the PSK for authentication of peers

Configuration
#############
# Set domain name
# Generate RSA Keys ::
  crypto key generate rsa modulus 2048
# Repeat on other peer
# Display RSA Keys and copy to other peer (Key Data portion) ::
  show crypto key mypubkey rsa
# Copy peer key data to originalk router
# Phase 1
# Transform Set
# IPSEC SA Lifetime
# ISAKMP PRofile
  # Identity
  # Keyring
# IPSEC PRofile
  # Transfor set
  # isakmp prof
# Tunnel Interface
  # Source
  # Destination
  # IPSEC/IPv4 Mode
  # Tunnel protection profile

Notes
#####
# What happens if the Sig is correct but PSK is wrong?

IOS Site-To-Site with Hostname Authentication
---------------------------------------------

Configuration
#############
# Verify Interface
# Verify Routing
# Verify Reachability between peers
# Phase 1 policy
# IPSEC TS
# IPSEC SA Lifetime
# Crypto ACL
# Cryto Map
# Set Domain Name
# Configure device to use hostname for isakmp identify ::
  crypto isakmp identity hostname
# Keyring
# ISAKMP PRofile
  crypto isakmp profile <isakmp-prof-label>
    match identity host domain <domain-name>
    keyring <keyring-name
# Set Crypto map to use isakmp profile

Notes
#####
# Is this really authenticating with hostname, as the IPs are still set manually?

IOS Site-To-Site with IPv6
==========================

Configuration
=============
# Enable IPv6 routing ::
  ipv6 unicast-routing
# Configure Interfaces ::
  ipv6 add <ip> <cidr>
# Configure Routing ::
  ipv6 route ::/0 <gateway-ip>
# ISAKMP Policy
# IPSec Transform Set
# IPSEC SA Lifetime
# Keyring ::
  crypto keyring <label>
    pre-shared-key addresses ipv6 <ip> key <psk>
# ISAKMP Profile
  match identity address ipv6 <ip>
  keyring <label>
# IPSec Profile
  # transform set
  # isakmp profile
# tunnel Interface
  # ip address
  # source
  # destination
  # mode ipsec ipv6
  # protection profile
  
IOS Site-To-Site with IPv6 and IOS CA issued RSA Signatures
===========================================================

Configuration
=============
# Same CA Setup as IPv4 just substiture IP/URL for IPv6 equivilent
  * IPv6 IP should be specified in '[' and ']' for URL
# Tunnel Interface same but with Ipv6 adresses
  * Mode should be ipsec ipv6

IOS Site-To-Site with stateless failover
===========================================================

Hub Configuration
==================
# Configure LAN Interface
  * Set IP
  * Enable Interface
  * Configure HSRP
    standby <group> ip <ip>
    standby <group> priority <value>
    standby <group> preempt
    standby <group> track <interface-state or routing>
# Configure WAN Interface
  * Set IP
  * Enable Interface
# Configure Routing
# Phase 1 Policy
# PSK
# TS
# SA Lifetime
# Crypto ACL
# Crypto Map
# Bind Crypto Map to interface
# Repeat on other HA router

Spoke Configuration
====######=========
# Configure LAN Interface
  * Set IP
  * Enable Interface
# Configure WAN Interface
  * Set IP
  * Enable Interface
# Configure Routing
# Phase 1 Policy
# PSK for both Hub Peer IPs
# Configure ISAKMP Keepalive
# TS
# SA Lifetime
# Crypto ACL
# Crypto Map
  * Define default peer
  * Add secondary peers
# Bind Crypto Map to interface

Notes
#####
* Requires two routers at the head-end site. Routers will use a VIP

IOS Site-To-Site with stateful failover
===========================================================

Hub Configuration
==================
# Same Config as for stateless
# Configure WAN Inteface
  * Configure HSRP
# Configure Redudancy (repeat for both peers) ::
  redundancy inter-device
    scheme standby <key>
  ipc zone default
    association <id>
      protocol sctp
        local-port <port>
          local-ip <ip>
        remote-port <port>
          remote-ip <peer-ip>
# Reload Peer Router
# Configure usual VPN details
# Crypto Map
  * Enable redundancy ::
    crypto map <name> redudnacy <name> stateful
    
Spoke Configuration
###################
# Same configiration as for stateless
# No need to specify multiple peers, use the VIP instead

Verification
============
show redundancy inter-device
    
ISO Site-To-Site with NAT-T
===========================
* No special configuration required on VPN Peers

Notes
#####
* UDP 500 and UDP 4500 should be allowed through any filtering device


IOS Site-To-Site to ASA Firewall (8.0) with PSK
===============================================

IOS Router
##########
* Phase 1 Policy
* PSK
* TS
* SA Lifetime
* Crypto ACL
* Crypto Map
* Bind Crypto Map to Interface

ASA Firewall
############
* Phase 1 ::
    crypto isakmp policy <id>
      authentication pre-share
      encryption <algorithm>
      hash <algorithm>
      group <dh-group>
      lifetime <seconds>
* Key ::
    crypto isakmp key <psk> add <peer-ip>
* IPSEC Transform-set ??? - Are default supplied
    crypto ipsec transform-set <name> <enc> <hash>
* IPSEC SA Lifetime ::
    crypto ipsec security-association lifetime seconds <seconds>
* Crypto ACL ::
    access-list <id-or-name>  permit ip <local-lan> <local-mask> <remote-lan> <remote-mask>
* Crypto Map ::
    crypto map <name> <seq> set transform-set <ts-name>
    crypto map <name> <seq> set peer <peer-ip>
    crypto map <name> <seq> match address <acl-id-or-name>
* Bind Crypto Map to Interface ::
    crypto map <name> interface <ifname>
* enable IPSec on interface ::
    crypto isakmp enable <ifname>

Notes
#####
* Tunnel group is not required, where is getting settings from (default group policy)?

Vertification
#############
* IOS
  show crypto isakmp sa
  show crypto ipsec sa
  show crypto session
* ASA
  show crypto isakmp sa
  show crypto ipsec sa
  show vpn-sessiondb l2l
  

IOS Site-To-Site to ASA Firewall (8.4) with PSK
===============================================

IOS Configuration
#################
* Same as for ASA 8.0

ASA Configuration
#################
* Phase 1
  crypto ikev1 policy 1
    authentication pre-share
    encryption <algorithm>
    hash <algorithm>
    group <dh-group>
    lifetime <seconds>
* Define tunnel and key
  tunnel-group <peer-ip> type ipsec-l2l
  tunnel-group <peer-ip> ipsec-attributes
    ikev1 pre-shared-key <psk>
* IPSEC Transform-set
  crypto ipsec ikev1 transform-set <ts-name> <encryption> <hashing>
* IPSEC Lifetime
  crypto ipsec security-assocaition life sec <seconds>
* Crypto ACL
  access-list <id-or-name>  permit ip <local-lan> <local-mask> <remote-lan> <remote-mask>
* Crypto Map
    crypto map <name> <seq> set ikev1 transform-set <ts-name>
    crypto map <name> <seq> set peer <peer-ip>
    crypto map <name> <seq> match address <acl-id-or-name>
* Bind Crypto Map to Interface
    crypto map <name> interface <ifname>
* Enable IPSec on Interface
    crypto ikev1 enable <ifname>

Verification
############
* ASA
  show crypto ikev1 sa
  show crypto ipsec sa
  show vpn-sessiondb l2l




IOS Site-To-Site to ASA Firewall (8.4) with RSA
===============================================

IOS Configuration
#################
* Same configuration is in previous labs with RSA
* Generate Keys
* Authenticate
* Enrollment
* Configure VPN


ASA Configuration
=================
# Same config as previous, with following additionas
# Set Domain Name
# Generate keys ::
  crypto key generate rsa
# Configure trusting of CA issues certificates ::
  crypto ca trustpoint <ca-name>
    enrollment url <scep-url>
    revocation-check none
# Authenticate CA ::
  crypto ca authenticate <ca-name>
# Enroll with the CA ::
  crypto ca enroll <ca-name> ::
# Phase 1 Policy ::
  crypto ikev1 policy 1
    authentication rsa-sig
# Tunnel Group ::
  ikev1 trust-point <ca-name>
# Crypto Map ::
  crypto map <name> <seq> trustpoint <ca-name>
  
  
  

IOS Site-To-Site to ASA Firewall using IKEv2
============================================

IOS Configuration
#################
# IKEv2 Proposal ::
  crypto ikev2 proposal <ike-prop-name>
    encryption <algorithm1> <..algoritmX>
    integrity <algorithm1> <..algoritmX>
    group <dh-group>

# IKEv2 Policy ::
  crypto ikev2 policy <ike-pol-name>
    proposal <prop-name>
    
# Keyring ::
  crypto ikev2 keyring <keyring-name>
    peer <peer-name>
      pre-shared-key local <psk>
      pre-shared-key remote <psk>
      address <peer-ip>
      
# IKEv2 Profile ::
  crypto ikev2 profile <ike-prof-name>
    match identity remote address <peer-ip>
    authentication local pre-share
    authentication remote pre-share
    keyring local <keyring-name>
    
# Transform Set ::
  crypto ipsec transform-set <ts-name> <encryption> <hashing>
    mode tunnel
    
# Security Association Lifetime ::
  crypto ipsec security-association lifetime seconds <seconds>

# Crypto ACL ::
  access-list <crypto-acl-name>  permit ip <local-net> <local-mask> <remote-net> <remote-mask>
  
# Crypto Map ::
  crypto map <cm-name> <seq> ipsec-isakmp
    match address <crypto-acl-name>
    set transform-set <ts-name>
    set peer <peer-ip>
    ikev2-profile <ike-prof-name>
    ! Optional
    set security-association lifetime seconds <seconds>
    
# Bind Crypto Map to Interface ::
  interface <type> <slot/num>
    crypto map <cm-name>

ASA Configuration
#################
# IKEv2 Policy ::
  crypto ikev2 policy <ike-pol-name>
    encryption
    integrity
    group
    lifetime
# Tunnel-group (type and ipsec-attirbutes) ::
  tunnel-group <peer-ip> type ipsec-l2l
  tunnel-group <peer-ip> ipsec-attributes
    ikev2 local-authentication pre-shared-key <local-psk>
    ikev2 remote-authentication pre-shared-key <remote-psk>
# IKEv2 Proposal ::
  crypto ipsec ikev2 ipsec-proposal <ike-prop-name>
    protocol esp encryption <algorithm>
    protocol esp integrity <algorithm>
# IPSec SA Lifetime ::
  crypto ipsec security-association lifetime seconds <seconds>
# Crypto ACL ::
  access-list <crypto-acl-name>  permit ip <local-net> <local-mask> <remote-net> <remote-mask>
# Crtypto Map ::
  crypto map <name> <seq> set ikev2 ipsec-proposal <name>
  crypto map <name> <seq> set peer <peer-ip>
  crypto map <name> <seq> match address <crypto-acl-name>
  ! Optional
  crypto map <cm-name> <seq> set security-association lifetime seconds <seconds>

# Bind Crypto Map to Interface ::
  crypto map <cm-name> interface <ifname>
# Enable IKEv2 on the Interface ::
  crypto ikev2 enable <ifname>
  
Verification
############
* IOS Device ::
  show crypto ikev2 sa [detail
  
* ASA ::
  show crypto ikev2 sa
  show crypto ipsec sa
  
  
IOS VPN Configuration with CCP
==============================
# Bootstrap the IOS devices with IP addressing and authentication so that CCP can push the config
  # Configure LAN interface IP Addressing
  # Configure Telnet/SSH authentication
  # Configure Usernames (or AAA)
  
# Start CCP
# Inform CCP about devices
  * Username/Password
  * IP addressing
  * Secure protocols to use or not
# CCP will then discover the device
# For each device:
  # Under Security > VPN. Select Create a site-To-Site VPN then Select Step-By-Step
  # VPN Connection information:
    * Interface
    * Peer Identity
    * Authentication
  # Click Next
  # IKE Proposals, modify as approrpiate:
    * Add new proposal
  # Click Next
  # Traffic to Protect
    * Specify Local Network
    * Specify Remote Network
    * Can also specify ACL
  # Click Next
  # Click FInish
  # Can the either save the config or deliver to the the device

Notes
#####
* Creates nasty object names, just like ASDM
