##################
Secure Shell (SSH)
##################

General
=======

* Both versions generally use a host-specific key of 2048-bits.
* Servers that support both SSH Version 1 and 2, usually indicate this by
  advertising version 1.99 in their connection banner.
* Minimum of 768-bits for RSA Key

Version 1
=========

* No defined standard, considered proprietary
* Ciphers (3DES, Blowfish, DES)
* Authentication (?)
* Supports only RSA Keys
* Forward Security provided through an additional server key, of 768-bits. This
  is commonly regenerated (e.g. once per hour)
* Lack of strong mechanism for ensuring integrity of connection

Version 2
=========

* Initially RFC 4251
* Incompatible with version 1
* Minimum of 768-bits for RSA Key
* Supports DSA, ECDSA and RSA Keys
* Forward Security provided through Diffie-Hellman  (DH) Key Exchange
* Integrity provided through HMAC (HMAC-HD5, HMAC-SHA1, etc)
* Multiple supported authentication methods

  * Password
  * Public Key (at least DSA or RSA)
  * Keyboard Interactive / Generic Message Exchange Authentication
  * GSSAPI
