---
layout: post
title:  "The Prying Eyes of Your ISP"
date:   2015-03-30 21:34:00
categories: VPS VPN
---
It has long been known that one has to be careful when using public and 
semi-public WiFi due to the ease with which prying eyes can snoop on internet
traffic. Back in the fall
we learned that the two largest wireless carriers, Verizon and AT&T, have been 
[injecting tracking cookies](http://arstechnica.com/security/2014/10/verizon-wireless-injects-identifiers-link-its-users-to-web-requests/)
into the web streams of their users without their knowledge or consent, and that Comcast had been 
[injecting JavaScript-based](http://arstechnica.com/tech-policy/2014/09/why-comcasts-javascript-ad-injections-threaten-security-net-neutrality/)
ads into web pages for users of its Xfinity WiFi service. And it has now come 
to light that AT&T is 
[scanning the traffic](http://arstechnica.com/information-technology/2015/03/atts-plan-to-watch-your-web-browsing-and-what-you-can-do-about-it/)
of the users of its gigabit internet service - unless they pay an additional $30+
"privacy fee".

Such behavior is expected when using the free services 
of the Facebooks and Twitters of the world. After all, when something is free 
that means you're the product. However, there is a certain expectation of 
privacy that comes with using a paid-for service, particularly a paid-for
*communications* service. Unfortunately, the laws that guarantee a measure
of privacy with voice communications do not extend to internet communications
 - at least, not yet. Perhaps, now that the FCC has reclassified ISPs as common
carriers, that will change. Until that changes, and perhaps even afterward, 
the only privacy you can count on is privacy you provide for yourself.

The simplest form of protection, at least for web browsing, is to use the
encrypted HTTPS protocol, instead of the plain-text HTTP protocol, wherever 
it is available. Unfortunately, HTTPS isn't offered everywhere. But even if
every nook and cranny of the internet was available with HTTPS, the protocol
still would only offer a modest amount of privacy. For one, the source and
destination of each packet are still visible for the world to see. This is a
fundamental part of how the internet works, so it won't be changing
anytime soon. Additionally, the protocol relies on the exchange of security
certificates that are vouched for by trusted agents, known
as Certificate Authorities, or CAs. A well-known weakness with all SSL-based
protocols, such as HTTPS, involves an attacker inserting themselves in between
the server and client during the certificate exchange such that the attacker
can then impersonate each endpoint. This is known as a 
[man-in-the-middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack),
or MitM, attack and ISPs are in the ideal position to institute such an attack.

A better solution is to use what is called a Virtual Private Network, or VPN. 
A VPN creates a virtual tunnel between client devices connected to the internet
that makes each client appear to be on the same local network. VPNs can be
configured to either route only client-to-client traffic or all of a client's
internet traffic through the tunnel. This second option is particularly 
interesting as a solution to problem of prying ISPs. This is because it allows us to
set up a tunnel between the machine we are using, be it laptop, smart device, 
or whatever, and a trusted machine on a trusted network. Furthermore, if this
tunnel is encrypted, then all the local ISP and anyone on the local network
can see is a stream of seemingly-random data between your client and the 
trusted machine.

One obvious hole in this scheme is this: your traffic must eventually be
exposed to the rest of the world where prying eyes can, well, pry, and where
you run the risk of a nefarious party pulling a MitM. To this, I pose the 
following: which do you distrust more, your ISP or a VPN hosting service?

Not only are there a number of VPN providers in the wild, but it is reasonably
simple to set up your own VPN using a virtual private server, or VPS. Contrast
that with the fact that most of us only have one, maybe two, ISPs to choose 
from. So, to rephrase my question: which do you distrust more, your ISP,
which considers you a captive audience who should be grateful for the 
overpriced service that they deign to provide you, or a VPN or VPS hosting
service, who is competing on price, availability, reliability, respectability,
and speed, among other parameters? 

... Yeah, me too. So, how about we set ourselves up a VPN? Since using an
existing VPN service is the easy way out, I will instead walk you through
setting up your own VPN on a VPS. Running your own VPN has a handful of benefits:

* Once set up, you can easily move your VPS to another service without a lot of
fuss
* You have greater control over what is happening to your traffic on the other
side of the tunnel
* It'll be fun and a good learning experience
* Plus, you can use your VPS for other things, such as an rss2email or personal
git server

So, stay tuned!
