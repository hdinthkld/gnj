$$$$$$$$$$$$
IOS SSL VPNs
$$$$$$$$$$$$

Components
==========

SSL VPN Gateway
---------------
Acts as the proxy for a connection to protected resources, it holds the following configuration details:
  * IP Address for gateway to listen on
  * Port Number
  * Supporting Cryptographic details (e.g. Encryption and Integrity algorithms)
  * Certificates to serve to client (trustpoint)
  
The Gateway is initially defined by the '** webvpn gateway <gw-name> **' command syntax

SSL VPN Context
---------------
Allows the gateway configuration to service multiple client types be presented differently depending on the context/domain being accessed:
  * Specifies the gateway on which it is listening
  * Authentication method to use
  * Policy group settings to use
  * User portal customisations
  * Defining what URLs users can access (webacl)
  * Limits on number of users than can access the portal
  * URL Masking
  
The Context is initially defined by the '** webvpn context <ctx-name> **' command syntax
  
SSL VPN Policy Group
--------------------
Included within the context configuration and specifies the resources that a user who is a member of the group should have access to:
  * Presentation of the SSL VPN Portal
  * File Access servers that the user can access
  * Port Forwarding List for use with Thin-Client applications (Requires Java Support)
  * Idle/Session timers
  * A permitted URL list (if different from the one defined for the general context)
  * Whether the user should be able to type in URLs manually or not
  
 A polocy group is defined with a previously created '** webvpn context **' and uses the '** policy group <name> **' command synax
 

Reference Material
==================
  * SSL VPN Configuration Guide (IOS 15M&T)
  http://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_conn_sslvpn/configuration/15-mt/sec-conn-sslvpn-15-mt-book/sec-conn-sslvpn-ssl-vpn.html#GUID-42CF964F-D399-49DA-A097-C2C93B705A10
  
  * SSL VPN Remote User Guide (for IOS 12.4 and below)
  http://www.cisco.com/c/en/us/td/docs/ios/12_4t/12_4t11/ht_sslug.html
  
  * AnyConnect Secure Mobility Client Adinistrator Guide (Release  4.4)
  http://www.cisco.com/c/en/us/td/docs/security/vpn_client/anyconnect/anyconnect44/administration/guide/b_AnyConnect_Administrator_Guide_4-4.html

IOS Clientless SSL Mode
=======================

Configuration
-------------

# Ensure all interfaces and routing is configured
# Set hostname and domain name
# Setup AAA / Local Usernames
# Generate Encryption Keys
  crypto key generate rsa modulus <bits>
# Enable HTTP Server
ip http server
ip http secure-server
# Set what IP and port the web gateway should listen on
webvpn gateway <gw-name>
  ip address <ip> <port>
  http-redirect port <port-to-redirect>

# Define Bookmark list 
  webvpn context <ctx-name>
    url-list <name>
      heading <"section heading">
      url-text <"URL Friendly Name"> url-value <url>

  policy group <policy-name>
    url-list <list-name>
    functions [file-access | file-browse | file-entry]

  default-group-polocy <policy-name>
  gateway <gw-name>
  inservice

Testing
-------
# Use client browser


IOS Thin-Client SSL Mode
=======================

Configuration
-------------

# Ensure all interfaces and routing is configured
# Set hostname and domain name
# Setup AAA / Local Usernames
# Generate Encryption Keys
  ::
  crypto key generate rsa modulus <bits>
# Enable HTTP Server
# Set what IP and port the web gateway should listen on
  ::
  webvpn gateway <gw-name>
    ip address <ip> <port>
    ! Redirect 80 to 443
    http-redirect port 80
    ! Start the web gateway
    inservice

# Define Bookmark list 
  ::
  webvpn context <ctx-name>
    port-forward <port-list>
      local-port <local-port> remote-server <ip> report-port <port> description <description>

    policy group <policy-name>
      port-forward<port-list>
      ! Hide the URL Bar
      hide-url-bar

    default-group-polocy <policy-name>
    gateway <gw-name>
    inservice


Testing
-------
# Connect to web gateway via web browser
# Click "Start" for Thin Client Applications
# Run Java Applet that appears in popup
# Window will provide information on the allowed ports
# Use client application to connect to local port using localhost or 127.0.0.1 as the IP

IOS Thick-Client SSL Mode
=======================
Prerequisites
# Java/ActiveX installed on Client
# Anyconnect/SVC package available to upload to router

Configuration
-------------
# Ensure all interfaces and routing is configured
# Set hostname and domain name
# Setup AAA / Local Usernames
# Generate Encryption Keys
  crypto key generate rsa modulus <bits>
# Enable HTTP Server
# Upload client package to IOS Device (unless preloading)
  * Usually takes the format of 'svcx.y.pkg' 
  ::
  copy ftp:<url-src> flash:<svc-file-path>

# Create Address pool
  ::
  ip local pool <pool-name> <start-ip> <end-ip>

# Configure Gateway
  ::
  webvpn gateway <gw-name>
    ip address <ip> port <port>
    http-redurect <port>
    inservice

# Configure Context
  ::
  webvpn context <name>
   policy group <policy-name>
     functions svc-enabled
     svc keep-client-installed
     svc default-domain <dns-domain>
     svc address-pool <ip-pool-name>
   
   gateway <gw-name>
   default-group-policy <policy-name>
   inservice

# Specify Installer packages to provide to users
  ::
  webvpn install svc flash:<svc-file-path>

# Create Loopback interface
  * Why is this needed - For advertising into routing protocol perhaps?
  ::
  int loopback <id>
    ip address <ip-in-subnet-of-local-pool> <mask>

Testing
-------
# Login to web interface via browser
# Click "Start" besides Tunnel Connection which will initiate the client installation

IOS SSL VPN with AAA
=======================

Prerequisites
#############
# Package installed on Flash
# User accounts configured on RADIUS Server
# RADIUS Client configured on RADIUS Server

Configuration
##############
# Ensure all interfaces and routing is configured
# Set hostname and domain name

# Create Local IP Address Pool
  ::
  ip local pool <pool-name> <start-ip> <end-ip>

# Enable AAA
  ::
  aaa new-model
  aaa authentication login <list-name> group radius

# Define Radius Server
  ::
  radius-server host <server-ip> timeout <seconds> key <key>

# Define Gateway
  ::
  webvpn gateway <gw-name>
    ip address <ip> port <port>
    http-redirect port 80
    inservice

# Create webvpn context and define any URL or port forwarding as required 
  ::
  webvpn context <ctx-name>
    ! heading / url-text
    ! port-forward
    policy group <policy-name>
      port-forward <port-list>
      url-list <url-list>
      functions [file-access | file-browse | file-entry | svc-enabled]
      svc address-pool <pool-name>
      
      ! Optional
      hide-url-bar
      svc keep-client-installed
      svc default-domain <domain-name>

    default-group-policy <policy-name>      
    gateway <gw-name>
      inservice

    ! Setup the authentication method to use
    aaa authentication list <list-name>

# Permit installation of client package through web interface 
  ::
  webvpn install svc flash:<svc-file-path>

Testing
##############

# From Router CLI
  ::
  test aaa group <group-name> <username> <password> legacy


High Availability Options
=========================
#  HSRP Stateless Failover (Active/Standby)
   * Multiple WebVPN groups per context, failover between two devices
   * Use HSRP to provide failover between devices

Multiple Contexts on single Gateway
===================================
# Configure as per other single gateway

# Define contexts
  ::
  webvpn context <ctx-name>
    ! url-list
    ! port-forward
    ! banner
    ! group policies
    default-group-policy <policy-name>
    gateway <name> domain <logical-domain>
    inservice
    
# Repeat for other contexts


Testing
#######
#  Access from a web browser using the context name in the URL (https://url/<logical-domain>)

SSL IOS VPN Advanced Features
=============================

Split-Tunnelling
----------------
Allows for only certain traffic to be sent over VPN or to exclude specific traffic from the VPN
  ::
  webvpn context <ctx-name>
    policy group <name>
      svc split [include | exclude] <ip-or-subnet> <mask>

Banner
-------
Displays a mesage to the user when the login the SSL VPN Service, can be used for following purposes
  * Show reference to Acceptable Use Policy / Monitoring Notification
  * Notifcation of scheduled maintenance
  * Confirming which gateway a user has contected to (e.g. in a HA scenerio)
  ::
  webvpn context <ctx-name>
    banner <banner-text>

URL-Filtering
-------------
Prevents users from being able to go to any URL they want either via the bookmarks or by manually typing in (most useful).

Any bookmarks that are defined but not allowed, will still show up but a message will be displayed indicating not assible

Can be combined with time range to specify any bookmarks that should only be accessible at certain times.

::
  webvpn context <ctx-name>
   time-range <tr-name>
     periodic daily <start-time> to <end-time>
     acl webacl
     permit url <url> time-range <tr-name>
   policy group <policy-name>
   acl webacl

Group-Lock
----------

Configuration
#############

webvpn context <ctx-name>
  aaa authentication domain @<ctx-name>
  

# Define users for specific group (could also be done via AAA attribute)
  username <user>@<ctx-name> password <password>

Testing
#######
Login as user specifying the username only, don't include the context name

Configuration
#############

General Troubleshooting
=======================

Commands
--------
  ::
  show webvpn stats
  show webvpn gateway
  show webvpn context [<ctx-name>]
  show webvpn policy group <policy-name> context {<ctx-name> | all}
  show webvpn session [user <username>] 
  show webvpn session [context <ctx-name>]
  show webvpn stats [detail]
  debug webvpn sso
  
  