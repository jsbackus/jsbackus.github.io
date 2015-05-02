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
transport protocol, which generates an excessive amount of traffic. This is 
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
transport, and when used with the AES encryption algorithm, is most likely secure.
However, there are concerns that a certain three-letter organization may be
exploiting weaknesses in the initial key exchange part of the protocol to 
decrypt traffic. Additionally, L2TP/IPsec uses multiple and specific ports, 
which may complicate setup when used in an environment that involves 
[network address translation](https://en.wikipedia.org/wiki/Network_address_translation),
or NAT, which is very common on IPv4 networks. Even if the server you want to
connect to is not behind a firewall, many ISPs are now using NAT to preserve
their dwindling supply of IPv4 addresses. You may also find that something 
along the pipe may be filtering for one or more ports used by L2TP/IPsec.

So: L2TP/IPsec = not a bad choice, but could be better.

There is a another open source choice that, while not generally installed by 
default, is available for most platforms: OpenVPN. OpenVPN creates a tunnel 
via a SSL/TLS connection over either TCP or UDP. The port number is 
completely configurable. In fact, you can even use port TCP 443, making your 
VPN traffic indistinguishable from HTTPS traffic - though I would avoid TCP
for the reasons stated above. OpenVPN supports all of the encryption algorithms
that OpenSSL supports, including AES, so it is as secure as L2TP/IPsec, if not
more so. OpenVPN does have an efficiency advantage of L2TP/IPsec as it has one
less layer of encapsulation. Plus, it supports compressing traffic before
encapsulating it, which should improve speed in some conditions, such as
web browsing.

So: OpenVPN = the better choice.

For more information on the difference choices:

  * <http://www.howtogeek.com/211329/which-is-the-best-vpn-protocol-pptp-vs.-openvpn-vs.-l2tpipsec-vs.-sstp>
  * <https://www.ivpn.net/pptp-vs-l2tp-vs-openvpn>

### Generating Keys ###
Before we can begin setting up OpenVPN itself, we need to generate a public 
certificate and private key for each of the server and each client.
This is best done on a private machine and *not* on the server, in order to
protect the private keys from being obtained by a nefarious party. The OpenVPN
(HowTo)[https://openvpn.net/index.php/open-source/documentation/howto.html] 
has a good write-up on this process, complete with explanations of each step.
Therefore, I'll just briefly go over the steps here using information specific 
to Fedora.

#### Setting Up easy-rsa ####
easy-rsa is a set of scripts for managing a simple certificate authority (CA)
based on OpenSSL. We need to install it on the machine that we are going to use
to generate our certificate/key pairs. This machine will be referred to as our
CA machine from here on out. To install it on a Fedora machine, simply:
{% highlight bash %}
sudo yum install -y easy-rsa
{% endhighlight %}

Once easy-rsa is installed, we need to set up our "CA database". Make a 
directory somewhere safe, such as a subdirectory off of your Documents, and
change into this directory.

In this directory, we need to create a symbolic link to all of the files in
/usr/share/easy-rsa/2.0/, except for vars, which we want to copy and edit. So:
{% highlight bash %}
cp /usr/share/easy-rsa/2.0/vars .
ln -s /usr/share/easy-rsa/2.0/* .
{% endhighlight %}

Now, edit vars in your favorite text editor. We can leave most of these 
settings as is. The only things I would change are the following:

* KEY_SIZE - make sure it is at least 2048. The larger the key, the more it
will slow down initial TLS negotiation but the more secure it will be. Since
we only perform the negotiation once per connection, it is in the mud. So, go
for bigger.
* These end up in the certificate, so change them to make sense, but their 
value doesn't matter from a functional point of view. Don't leave them 
blank:
  * KEY_COUNTRY
  * KEY_PROVINCE
  * KEY_CITY
  * KEY_ORG
  * KEY_EMAIL
  * KEY_OU

#### Initialize the CA ####
Now that we have easy-rsa set up, we need to establish the certificate 
database. Type the following in your terminal:
{% highlight bash %}
source vars
./clean-all
./build-ca
{% endhighlight %}

The last command, build-ca, will run for a bit and then prompt you for some
information for the CA cert and key. For the most part, keep the defaults as
they are what you entered in vars. The one you need to change is "Common Name".
Enter something unique that indicates to you that it refers to your certificate
authority, such as "My CA".

This step generates two files:

* ca.crt - public certificate which is copied to the server and all clients
* ca.key - private key (absolutely, positively, keep this secret!)

#### The Server Cert and Key ####
Generating the server key and certificate is very similar to generating the
CA certificate. Start the process with:
{% highlight bash %}
./build-key-server server
{% endhighlight %}

If you properly set up the vars file, you should be able to use the default
for all of the prompts. The "Common Name" field should be populated with the
argument you provided to the build-key-server script. For this step, use 
server.

One of the new questions you are prompted with is whether or not to 
specify a challenge password. Just leave this one blank. 
Two additional new prompts are whether or not to sign the certificate and 
whether or not to commit the signed certificate to the database. Answer 'y' 
in both cases.

This step generates two files for the server:

* server.crt - public certificate
* server.key - private key (keep it secret!)

#### Diffie-Hellman Parameters ####
You also need to generate 
(Diffie-Hellman)[http://mathworld.wolfram.com/Diffie-HellmanProtocol.html] 
parameters for the server. Do the following in your terminal:
{% highlight bash %}
./build-dh
{% endhighlight %}

This step generates a file, dh<N>.pem, where <N> = KEY_SIZE in vars. In our
example we should get dh2048.pem.

#### The Client Certs and Keys ####
Now we must generate a key and certificate pair for each client. The process is
almost identical to generating the server certificate with the following 
differences:

1. The script name is build-key, not build-key-server
1. Specify the client's name (without any spaces) at each invocation.

For example, to generate a key/cert pair for "My Awesome Phone", do:
{% highlight bash %}
./build-key MyAwesomePhone
{% endhighlight %}

Again, you should be able to use the defaults and the "Common Name" field will 
be populated by the argument provided to build-key. Be sure to sign and commit 
the certificate. Repeat this step for each client.

This step will generate two files for each client, where ClientName is the 
argument provided to build-key:

* ClientName.crt - public certificate
* ClientName.key - private key (keep it secret!)

When you want to add a new client you will need to repeat this step, however,
you will need to re-source the vars file. For example, say two months after
setting up your VPN your spouse, Pat, gets a new tablet and you need to add it
to the VPN. Change to your CA directory and do the following:
{% highlight bash %}
source vars
./build-key PatsShinyNewTablet
{% endhighlight %}

#### tls-auth Shared-Secret Key ####
The last key we want to generate is a shared-secret key for use with tls-auth.
This step helps protect our VPN against DoS attacks and port scans because it
adds an additional signature to all SSL/TLS handshake packets.
This step does not strictly have to happen on the CA, but since it does
involve generating a key that needs to be copied to the server and all clients,
it has been included here. This step does require openvpn, so install it with:
{% highlight bash %}
sudo yum install -y openvpn
{% endhighlight %}

To generate the shared-secret key file, ta.key, type the following into your
terminal:
{% highlight bash %}
openvpn --genkey --secret ta.key
{% endhighlight %}

This file must be given to the server and all clients.

#### Distributing Certs and Keys ####
This step requires care, so as to not expose files that need to be kept secret.
Each machine gets certain files, which should be distributed over as secure a 
channel as you can muster - i.e. home network or thumb drive. 

Here are the files to send to each machine:

* Server:
  * ca.crt
  * server.key
  * server.crt
  * ta.key
* ClientN:
  * ca.crt
  * ClientN.key
  * ClientN.crt
  * ta.key

Make sure to keep a copy in your CA database directory - and (securely!) back 
that directory up!

### Setting Up the Server ###
The trickiest part of setting up OpenVPN is setting up the server.

dd

### Setting Up Clients ###
dd

### Additional Considerations ###
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
  * Sample configs?
* DNS
  * <https://www.opendns.com/premium-dns/>
  * <https://developers.google.com/speed/public-dns/>
  * <http://pcsupport.about.com/od/tipstricks/a/free-public-dns-servers.htm>
