##############################################
Cisco Anyconnect Clientless IOS Implementation
##############################################

TBC

Configuration Steps
===================

#. Generate RSA Keys for use with SSL/TLS
#. Enable AAA
#. Enable the HTTP/HTTPS Server
#. Configure the WebVPN Gateway
#. Configure the WebVPN Context
#. Configure Local Accounts

Generate RSA Keys
-----------------

In order for the HTTP Server to also provide SSL/TLS services we need to
generate the public and private keys that it will use:

.. code-block:: none

  crypto key generate rsa modulus <bit-size>

Enable AAA
----------

So that users can be authenticated and authorised, its necessary to enable
the AAA sub-system and optionally different a method list specifying how
users should be authenticated.

.. code-block:: none

  ! Enable the AAA sub-system
  aaa new-model

  ! Optional - Define a specific method list for WebVPN authentication
  aaa authentication login <webvpn-method-list-name> local



Enable the HTTP/HTTPS Server
----------------------------

In order for users to be able to access the SSL Portal, an HTTP/HTTPS server
must be enabled on the IOS router as follows:

.. code-block:: none

  ip http server
  ip http secure-server


Configure the WebVPN Gateway
----------------------------

To accomplish this the following information is needed:

* IP Address
* Port Number

.. code-block:: none

  webvpn gateway <web-gw-name>
    ip address <web-gw-ip> port <web-gw-port>

    ! Optional - Configure gateway hostname, should match certificate used
    hostname <web-gw-hostname>

    ! Optional - Redirect HTTP to HTTPS
    http-redirect <http-port>

    ! Optional - Specify supported cipher suites
    ssl encryption [aes-sha1] [3des-sha1] [rc4-md5]

    ! Enable the gateway
    inservice


Configure the WebVPN Context
----------------------------

A WebVPN context is a means by which a certain portal type can be displayed to
users.  The context can either be specified by the user or provided as part
of the users authorization properties (e.g. from RADIUS).

.. code-block:: none

  webvpn context <web-cxt-name>

    ! Optional Specify how users should be authenticated, global config will
    ! be used if not specified
    aaa authentication list <webvpn-method-list-name>

    ! Create a policy for this context (multiples can exist)
    policy group <webvpn-policy-group-name>
      ! All the below settings are optional

      banner <login banner string>
      hide-url-bar
      port-forward <port-list-name>
      timeout [idle <seconds] [session <seconds>]
      url-list <url-list-name>

    ! Specifies the default policy to use if nothing is specified from AAA
    default-group-policy <web-vpn-policy-group-name

    ! Associates this context with a specifiic SSL VPN gateway
    gateway <web-gw-name> [domain <domain-name>]

    ! Optional - Display a message on the login screen
    login-message

    ! Optional - Maximum allowed users under this context
    max-users <no-of-users>

    ! Enable the context
    inservice

Configure Local Accounts
------------------------

If user accounts will not be configured on a central authentication server, it
is necessary to configure the users locally on the IOS router.

.. code-block:: none

  username <webvpn-username> <webvpn-password>

Troubleshooting
===============

The following commands can be useful in troubleshooting WebVPN

.. code-block:: none

  debug webvpn aaa
  debug aaa accounting


Reference Documents
===================

**SSL VPN Configuration Guide, Cisco IOS Release 15M&T**

http://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_conn_sslvpn/configuration/15-mt/sec-conn-sslvpn-15-mt-book/sec-conn-sslvpn-ssl-vpn.html

**SSL VPN Remote User Guide**

http://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_conn_sslvpn/configuration/15-mt/sec-conn-sslvpn-15-mt-book/sec-conn-sslvpn-remote.html
