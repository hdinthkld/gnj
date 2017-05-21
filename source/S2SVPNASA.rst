$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Site To Site VPNS with Cisco ASA
$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$

Implementing Site-To-Site VPNs on Cisco ASA
=============================================

Basic ASA IKEv1 Site-To-Site VPN ASDM Configuration
---------------------------------------------------
Requirements
############
# Java installed on management PC
# Following bootstrap configuration on the ASA
  * IP Addressing for management interface
  * Routing to management PC
  * Username/Password configured
  * HTTP Server enabled and acess granted from the Management PC
  * ASDM Image copied to ASA Flash and enabled
  

Configuration
#############
# Start ASDM and login
# Select Configuration
# Navigate to 

Notes
#####
* Normal to receive certificate error when accessing as ASA is using self-signed certificate initially
* Wizard also available, Wizards > VPN Wizards > Site-To-Site VPN Wizard


Basic ASA IKEv1 Site-To-Site VPN CLI Configuration
--------------------------------------------------
# Configure Phase 1 Policy ::
  * For ASA less than 8.4.1 ::
      crypto isakmp policy <priority>
        encryption <algorithm>
        hash <algorithm>
        group <dh-group>
        lifetime <seconds>
        authentication pre-share
  
  * For later ASA versions ::
      crypto ikev1 policy <priority>
        encryption <algorithm>
        hash <algorithm>
        group <dh-group>
        lifetime <seconds>
        authentication pre-share

# Configure PSK (<= v8.0) ::
  crypto isakmp key <key> address <peer-ip>
 
# Configure IPSec Transform-Set
  * For ASA less than 8.4.1 ::
      crypto ipsec transform-set <ts-name> <encryption-algorithm> <integrity-alogorithm>
      
  * For later ASA versions ::
      crypto ipsec ikev1 transform-set <ts-name> <encryption-algorithm> <integrity-alogorithm>
 
# Configure IPSec SA Lifetime ::
  crypto ipsec security-association lifetime seconds <seconds>
  
# Configure Encryption Domain (Crypto ACL) ::
  access-list <acl-name> permit ip <local-net> <local-mask> <remote-net> <remote-mask>
  
# Configure Crypto Map ::
  crypto map <cm-name> <seq> match address <acl-name>
  crypto map <cm-name> <seq> set transform-set <ts-name>
  crypto map <cm-name> <seq> set peer <peer-ip>
  ! optional - override default value
  crypto map <cm-name> <seq> set security-association lifetime seconds <seconds>
  
# Configure Connection Profile (Tunnel-group> ::
  * For ASA >= 7.0 and less than 8.4.1 ::
      tunnel-group <peer-ip> type ipsec-l2l
      tunnel-group <peer-ip> ipsec-attributes
        pre-shared-key <key>
  * For ASA later versions ::
      tunnel-group <peer-ip> type ipsec-l2l
      tunnel-group <peer-ip> ipsec-attributes
        ikev1 pre-shared-key <key>

# Enable ISAKMP on the approropriate (e.g. Internet facing) interface ::
  crypto map <cm-name> interface <ifname>

# Enable ISAKMP ::
  * For ASA less than 8.4.1 ::
      crypto isakmp enable <ifname>
  * For later ASA versions ::
      crypto ikev1 enable <ifname>
  
  
Basic ASA IKEv1 VPN with RSA
------------------
# Define hostname ::
  hostname <hostname>

# Define domain name ::
  domain-name <domain-name>

# Configure Trusted CA ::
  crypto ca trustpoint <ca-name>
    enrollment url http://<url>
    
# Download CA certificates and accept them ::
  crypto ca authentication <ca-name>

# Enroll with the CA ::
  crypto ca enroll <ca-name>

# Configure Phase 1 Policy ::
  crypto isakmp policy <priority>
    encryption <algorithm>
    hash <algorithm>
    group <dh-group>
    lifetime <seconds>
    authentication rsa-sig

# Configure IPSec Transform-Set
  * For ASA less than 8.4.1 ::
      crypto ipsec transform-set <ts-name> <encryption-algorithm> <integrity-alogorithm>
      
  * For later ASA versions ::
      crypto ipsec ikev1 transform-set <ts-name> <encryption-algorithm> <integrity-alogorithm>
  
# Configure IPSec Transform-Set (>= v8.4.1) ::
  
  
# Configure IPSec SA Lifetime ::
  crypto ipsec security-association lifetime seconds <seconds>

# Configure Encryption Domain ::
  access-list <acl-name> permit ip <local-net> <local-mask> <remote-net> <remote-mask>
  
# Configure Crypto Map ::
  crypto map <cm-name> <seq> match address <acl-name>
  crypto map <cm-name> <seq> set transform-set <ts-name>
  crypto map <cm-name> <seq> set peer <peer-ip>
  ! optional - override defalt value
  crypto map <cm-name> <seq> security-association lifetime seconds <seconds>

# Configure Connection Profile (Tunnel-group> ::
  * For ASA less than 8.4.1 ::
      tunnel-group <peer-ip> type ipsec-l2l
      tunnel-group <peer-ip> ipsec-attributes
        trustpoint <ca-name>
    
# Define interfaces on which to accept this VPN connection ::
  crypto map <cm-name> interface <ifname>

# Enable ISAKMP ::
  * For ASA less than 8.4.1 ::
      crypto isakmp enable <ifname>
  * For later ASA versions ::
      crypto ikev1 enable <ifname>






Based ASA IKEv2 VPN with PSK
----------------------------
# Create IKEv2 Proposal ::
  crypto ikev2 policy <seq>
    encryption <algorithm>
    integrity <algorithm>
    group <dh-group>
    lifetime <seconds>
    authentication pre-share

# Create IPSEC Transform Set ::
  crypto ipsec ike2 ipsec proposal <ikev2-proposal-name>
    protocol esp integrity <algorithm>
    protocol esp encryption <algorithm>

# Define global IPSec SA Lifetime ::
  crypto ipsec security-association lifetime seconds <seconds>

# Define Connection Profile ::
  tunnel-group <peer-ip> type ipsec-l2l2
    ike21 local-authentication pre-shared-key <local-key>
    ikev2 remote-authentication pre-shared-key <remote-key>
# Define Encryption Domain ::
  access-list <crypto-acl> permit ip <local-net> <local-mask> <remote-net> <remote-mask>
# Crypto map ::
  crypto map <cm-name> <seq> ipsec-isakmp
  crypto map <cm-name> <seq> set ikev2 ipsec-proposal <ikev2-proposal-name>
  crypto map <cm-name> <seq> set peer <peer-ip>
  crypto map <cm-name> <seq> match address <crypto-acl>

# Define interface from which to accept these VPN connections
  crypto map <cm-name> interface <ifname>

# Enable IKEv2 on the interface
  crypto ikev2 enable <ifname>


Based ASA IKEv2 VPN with PSK
----------------------------
Prequistes
##########
* Ensure hostname is set
* Ensure domain name is set
* Ensure time is correct

Configuration
#############

# Define the Trusted CA ::
  crypto ca trustpoint <ca-name>
    enrollment url http://<ca-url>

# Download CA certificates, verify the given Hash is correct ::
  crypto ca authenticate <ca-name>
  
# Request certificate from the CA (Enrollment) ::
  crypto ca enrol <ca-name>

# Create IKEv2 Proposal ::
  crypto ikev2 policy <seq>
    encryption <algorithm>
    integrity <algorithm>
    group <dh-group>
    lifetime <seconds>
    authentication rsa-sig

# Create IPSEC Transform Set ::
  crypto ipsec ike2 ipsec proposal <ikev2-proposal-name>
    protocol esp integrity <algorithm>
    protocol esp encryption <algorithm>

# Define global IPSec SA Lifetime ::
  crypto ipsec security-association lifetime seconds <seconds>

# Define Connection Profile ::
  tunnel-group <peer-ip> type ipsec-l2l2
    ikev2 local-authentication certificate
    ikev2 remote-authentication certificate
    
# Define Encryption Domain ::
  access-list <crypto-acl> permit ip <local-net> <local-mask> <remote-net> <remote-mask>
  
# Crypto map ::
  crypto map <cm-name> <seq> ipsec-isakmp
  crypto map <cm-name> <seq> set ikev2 ipsec-proposal <ikev2-proposal-name>
  crypto map <cm-name> <seq> set peer <peer-ip>
  crypto map <cm-name> <seq> set trustpoint <ca-name>
  crypto map <cm-name> <seq> match address <crypto-acl>

# Define interface from which to accept these VPN connections ::
  crypto map <cm-name> interface <ifname>

# Enable IKEv2 on the interface ::
  crypto ikev2 enable <ifname>


ASA VPN setup with IP SLA
-------------------------
# Requirements
  * Configure IP SPA
  

# Configure ICMP SLA ::
  sla monitor <sla-id>
    type echo protocol ipIcmpEcho <ip> interface <dst-int>
    timeout <ms>
    frequency <sec>
  
  sla monitor scheudle <sla-id> start-time now life forever
  
# Check Track Status ::
  show track <id>
  
# ISAKMP Policy
# PSK for both peers
# ISAKMP Keepalive
# IPSEC Transform-set
# IPSEC SA Lifetime
# Crypto ACL
# Crypto Map
  * Define multiple peers
# Define Map on both external interfaces
# Enable ISAKMP on both interfaces

