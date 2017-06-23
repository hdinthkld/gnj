.. _cisco_macsec:

############
Cisco MACSec
############

Overview
########

MACSec (IEEE 802.1AE) is a layer 2 encryption specification to provide wire-rate encryption at gigabit speeds. Providing both confidentiality and integrity of all communications over the link.

Often MACSec is combinded with other technologies such as 802.1X in order to provide additioal identification of users/devices on the network.  Theres not point spending resources encrypting traffic that is to/from an unauthorised user/device.

MACSec can provide protection against man in the middle or "shadow" uses who snoop the wire between the end user and the network switch.

AES-GCM is used as the authenticated encryption algorithm using a 128-bit symmetric key for encryption and decryption.

GMAC authenticates each packet.

MKA MACsec encryption is used for host-to-switch encryption

Either SAP or NDAC are used for Switch to Switch encyption. These cannot be used
alongside Cisco NEAT which intended for compact switches.

Further Reading
###############

For more details refer to the following external documentation:

Catalyst 4500 Series Switch Software Configuration Guide, Release IOS XE 3.3.0SG and IOS 15.1(1)SG [c7]_
