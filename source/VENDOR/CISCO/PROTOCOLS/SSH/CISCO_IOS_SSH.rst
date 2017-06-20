###############################
SSH Implementation on Cisco IOS
###############################


Cisco SSH Limitations
===========

* Cisco SSH Client cannot propose public key authentication method.

Cisco SSH Version 2 Enhancements
================================

.. note:: Cisco SSH implementations of version 2 require a miminum key modulus
          size of 768-bits.

* VRF-Aware
* DH Group Exchange
* Supports Keyboard-Interactive (RFC 4256) and Password-Based authentication
* Supports RSA-based public key authentiation for client/server
* SSH debug enhancements

External References
==================

.. rubric:: Secure Shell (Wikipedia)

https://en.wikipedia.org/wiki/Secure_Shell

.. rubric:: Secure Shell Configuration Guide, Cisco IOS Release 15S

http://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_usr_ssh/configuration/15-s/sec-usr-ssh-15-s-book/sec-secure-shell-v2.html#GUID-A906A0EE-3C84-4672-BD12-064A2A5BD7F2
