#########################################################################
Cisco CCNP - Implementing Cisco Secure Mobility Solutions (300-209 SIMOS)
#########################################################################

.. toctree::
   :titlesonly:
   :caption: Contents:

1.0  Secure Communications (32%)

  1.1 Site-to-site VPNs on routers and firewalls

    1.1.a  :ref:`Describe GETVPN <cisco_getvpn_overview>`

    1.1.b  Implement IPsec (with IKEv1 and :ref:`IKEv2 <cisco_ikev2_overview>`
    for both IPV4 & IPV6) using
    :ref:`IOS <ios_ikev1_legacy_cryptomap>` and
    :ref:`ASA <asa_ikev1_s2s_psk>`

    1.1.c  Implement DMVPN (hub-Spoke and spoke-spoke on both IPV4 & IPV6)

    1.1.d  Implement FlexVPN  (hub-Spoke on both IPV4 & IPV6) using local AAA

  1.2  Implement remote access VPNs

    1.2.a  Implement AnyConnect IKEv2 VPNs on ASA and routers

    1.2.b  Implement AnyConnect SSLVPN on ASA and routers

    1.2.c Implement clientless SSLVPN on ASA and routers

    1.2.d  Implement FLEX VPN on routers

2.0 Troubleshooting, Monitoring, and Reporting Tools (38%)

  2.1  Troubleshoot VPN using ASDM & CLI

    2.1.a  Troubleshoot IPsec

    2.1.b  Troubleshoot DMVPN

    2.1.c  Troubleshoot FlexVPN

    2.1.d  Troubleshoot AnyConnect IKEv2 and SSL VPNs on ASA and routers

    2.1.e  Troubleshoot clientless SSLVPN on ASA and routers

3.0 Secure Communications Architectures  (30%)

  3.1  Design site-to-site VPN solutions

    3.1.a Identify functional components of
    :ref:`GETVPN <cisco_getvpn_functional_components>`,
    :ref:`FlexVPN <cisco_flexvpn_functional_components>`,
    :ref:`DMVPN <cisco_dmvpn_functional_components>` and
    :ref:`IPsec <cisco_ipsec_functional_components>`

    3.1.b VPN technology considerations based on functional requirements

    3.1.c High availability considerations

    3.1.d Identify VPN technology based on configuration output

  3.2 Design remote access VPN solutions

    3.2.a Identify functional components of
    :ref:`FlexVPN <cisco_flexvpn_functional_components>`,
    :ref:`IPsec <cisco_ipsec_functional_components>` and
    :ref:`Clientless SSL <cisco_clssl_functional_components>`

    3.2.b VPN technology considerations based on functional requirements

    3.2.c High availability considerations

    3.2.d Identify VPN technology based on configuration output

    3.2.e :ref:`Identify AnyConnect client requirements <cisco_anyconnect_client_reqs>`

    3.2.f Clientless SSL browser and client considerations/requirements

    3.2.g Identify split tunneling requirements

  3.3 :ref:`Describe encryption, hashing, and Next Generation Encryption <nge_overview>`

    3.3.a Compare and contrast
    :ref:`Symmetric <crypto_symenc>` and
    :ref:`Asymmetric <crypto_asymenc>` key algorithms

    3.3.b Identify and describe the cryptographic process in VPNs:

    * :ref:`Diffie-Hellman <vpn_ipsec_dh>`
    * :ref:`IPsec <vpn_ipsec_intro>` – :ref:`ESP <vpn_ipsec_esp>`, :ref:`AH <vpn_ipsec_ah>`,
      :ref:`IKEv1 <vpn_ipsec_intro>`, :ref:`IKEv2 <vpn_ikev2_overview>`
    * :ref:`Hashing Algorithms MD5 and SHA <crypto_hashing_methods>`
    * :ref:`Authentication methods <vpn_ipsec_authmethods>`

    3.3.c :ref:`Describe PKI components and protection methods <pki_overview>`

    3.3.d :ref:`Describe Elliptic Curve Cryptography <crypto_ecc>`

    3.3.e Compare and contrast
    :ref:`SSL <ref_ssl_ssltls>`,
    :ref:`DTLS <ref_ssl_dtls>` and
    :ref:`TLS <ref_ssl_ssltls>`
