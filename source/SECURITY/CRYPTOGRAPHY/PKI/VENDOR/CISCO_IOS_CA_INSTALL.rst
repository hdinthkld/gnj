########################################
Setting Up Cisco IOS Router as CA Server
########################################

Configuration Steps
===================

.. rubric:: Pre-Requisites

1. Configure interface on which to service request
2. Configure approrpiate static/dynamic routing to reach requesting devices
3. Ensure time on the device is correct (NTP recommended)

.. rubric:: Generate the public/private keys

::

   crypto key generate rsa general-keys exporting label <CA-LABEL> modulus 2048

.. rubric:: Export the keys (Public and private)

::

  crypto key export rsa <label-name> pem url nvram: <encryption> <key>

.. rubric:: Enable HTTP Server for SCEP requests

::

 ip http server

.. rubric:: Create CA Server

::

 crypto pki server <db-name>
   database level minimum
   database url nvram:
   issuer-name <cn=xxxxx, c=xxxxx>
   lifetime certificate <days>
   grant [auto]
   no shutdown

.. note:: You will be prompted to enter a password to protect private key


Verification Steps
##################

.. rubric:: Verify server is running

::

 show crypto pki server
