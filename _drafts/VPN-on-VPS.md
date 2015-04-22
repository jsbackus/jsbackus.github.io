---
layout: post
title:  "Setting Up Your Own VPN on a VPS"
date:   2015-04-21 21:00:00
categories: VPS VPN
---

Now that we have our own VPS up and running, it is time for protect our 
packets. There are a number of open source VPN solutions available. We want
a solution that is available for smart devices as well as for PCs. The two
most common VPN solutions supported by most platforms right out of the box are
Point-to-Point Tunneling Protocol, or PPTP, and Layer 2 Tunnel Protocol over
IPSec, or L2TP/IPsec.

PPTP is the oldest of the two, and is probably supported on the most platforms
thanks to it being part of Windows for ages. While it does have some form of
encryption, this encryption is arguably little better than 
[ROT-13](https://en.wikipedia.org/wiki/Rot13). 

Additionally, it uses TCP as the
transport protocol, which generates excessive amount of traffic. This is 
because PPTP uses TCP to create a PPP tunnel, which is in turn used to carry IP
(i.e. TCP/IP) packets. Since TCP requires the receiver to acknowledge (ACK) all
packets it receives, and requires that the sender, must then ACK the ACK.
Because PPTP is
encapsulating TCP packets in PPP packets and transporting them with TCP, each
TCP packet for the initial TCP layer generates 3 TCP packets after 
encapsulation (DATA, ACK, and ACK-the-ACK) - which is *bad*.

So: PPTP = *bad*, for many reasons.

The L2TP itself isn't encrypted, which is why it is often coupled with IPsec.
IPsec is very similar to the IP transport protocol but with encryption added
in. For the most part, L2TP/IPsec is a good protocol. It uses UDP for 
transport, and when used with the AES encryption algorithm, is probably secure.
However, there are concerns that a certain three-letter organization may be
exploiting weaknesses in the initial key exchange part of the protocol to 
decrypt traffic. Additionally, L2TP/IPsec uses multiple and specific ports, 
which may complicate setup when used in an environment that involves 
[network address translation](https://en.wikipedia.org/wiki/Network_address_translation),
or NAT, which is very common on IPv4 networks. Even if the server you want to
connect to is not behind a firewall, many ISPs are now using NAT to preserve
their dwindling supply of IPv4 addresses. You may also find that something 
along the pipe may be filtering for one or more ports used by L2TP/IPsec.

So: L2TP/IPsec = not a bad choice.

There is a another open source choice that, while not generally installed by 
default, is available for most platforms: OpenVPN. OpenVPN creates a tunnel 
via a SSLv3/TLSv1 connection over either TCP or UDP. The port number is 
completely configurable. In fact, you can even use port TCP 443, making your 
VPN traffic indistinguishable from HTTPS traffic - though I would avoid TCP
for the reasons stated above. OpenVPN supports all of the encryption algorithms
that OpenSSL supports, including AES, so it is as secure as L2TP/IPsec, if not
more so. OpenVPN does have an efficiency advantage of L2TP/IPsec as it has one
less layer of encapsulation. Plus, it supports compressing traffic before
encapsulating it, which should improve speed in some conditions.

So: OpenVPN = the better choice.

For more information on the difference choices:

  * <http://www.howtogeek.com/211329/which-is-the-best-vpn-protocol-pptp-vs.-openvpn-vs.-l2tpipsec-vs.-sstp>
  * <https://www.ivpn.net/pptp-vs-l2tp-vs-openvpn>


dd

todo:

* Which VPN?
* OpenVPN
  * General:
    * <https://openvpn.net/index.php/open-source.html>
    * <https://openvpn.net/index.php/open-source/documentation/howto.html>
    * <https://community.openvpn.net/openvpn/wiki/Hardening>
    * Adding route for local network? Doesn't seem to be necessary...
  * Configure server:
    * Systemd:
    * FirewallD:
      * firewall-cmd --zone=external --add-service=openvpn
      * Set zone with: <http://forums.fedoraforum.org/showthread.php?t=289546>
  * Configure Client:
    * Systemd:
    * NetworkManager?
  * iOS
    * <https://docs.openvpn.net/docs/openvpn-connect/openvpn-connect-ios-faq.html>
    * <https://itunes.apple.com/us/app/openvpn-connect/id590379981?mt=8>
  * Android
    * <https://play.google.com/store/apps/details?id=net.openvpn.openvpn&hl=en>
    * <https://docs.openvpn.net/docs/openvpn-connect/openvpn-connect-android-faq.html>
  * Windows Phone?
    * Doesn't appear so. There is an OpenVPN connect app, which appears to be a scam.
* DNS
  * <https://www.opendns.com/premium-dns/>
  * <https://developers.google.com/speed/public-dns/>
  * <http://pcsupport.about.com/od/tipstricks/a/free-public-dns-servers.htm>
