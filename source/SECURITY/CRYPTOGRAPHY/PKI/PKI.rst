.. _pki_overview:

##################################
Public Key Infrastructure Overview
##################################

.. toctree::
   :titlesonly:

   VENDOR/VENDOR_IMPLEMENTATIONS



PKI Introduction
================

* Framework for managing the security attributes between peers who are enagaged in secure communication

PKI Message Echange Process
---------------------------

* Host generates RSA Signature (Public and private key) and sends public key to CA (CSR) for signing
* CA will sign the certificate request with it's private key, validating it origin in form of a certificate
* Host will save certificate and use as the public key portion in communicating with other peers
