---
layout: post
title:  "Setting Up Your VPS"
date:   2015-03-31 20:34:00
categories: VPS
---
Last time we talked about how a VPN can help protect our data and our privacy 
from would-be snoopers on public and semi-public networks, as well as from
our internet service providers. I proposed two options - using a VPN service
or setting up your own VPN on a virtual private server, or VPS. There is a
third option that I intentionally neglected to mention, which is to set up
a VPN on your own hardware. I generally dismiss this option for a few reasons.

For one, setting up a VPN on your own hardware will not protect you from your
ISP. Kinda negates one of the major reasons for using a VPN.

Even if you happen to trust your ISP, 
very few of us are lucky enough to have an even remotely symetric
connection. Most of us can get decent-to-good downstreams of 10+ Mbps, but 
we still have to contend with upstreams of 1Mbps or less. Why is this a 
problem? Well, say you are using a VPN out of your home to protect your 
laptop when you use it on a public network. All data you send and receive
has to go through the connection at your house. Since the VPN server in your
house is just a relay, it has to retransmit all data it receives, thus all
data to and from your laptop is limited to 1Mbps. Yikes!

Hosting the VPN on a woefully imbalanced connection made sense several years 
ago, before ISPs started mining their customers' data streams and before 
VPS-hosting services were so plentiful. Now, you can rent a VPS for less than
it cost to maintain and power your own equipment. For example, as of 4/4/2015
[Atlantic.net](https://www.atlantic.net) offers a Linux option with 256MB of
RAM, 1 virtual CPU, 10GB of storage, 1TB of outbound data, and unlimited
inbound data, all for $0.99 a month. While 256MB of RAM on a server is 
miniscule by today's standards, it is still enough to run a VPN with plenty
left over. Now the downside to a "Go Server", as they call it, is that it is
a flat rate for the month, unlike their larger servers which are billed only
when they are provisioned (i.e. "up") on a per-second basis.

So, in today's post we'll go through setting and securing our "baby server", 
leaving the details of actually setting up the VPN for a later post. Before
getting started, however, we need to make sure we have the necessary software.
For this step, we mainly need the SSH client and a key generator. For those on
Linux or OS X systems, these should be installed by default. You can verify
that they are installed by opening up a terminal and typing the following:
{% highlight ruby %}
which ssh
which ssh-keygen
{% endhighlight %}

For those on a Windows system, you will need to download 
[putty](http://www.putty.org/). I would go ahead and grab the installer 
that contains everything. You will definitely need putty.exe and puttygen.exe
for this part, and later you'll need pscp.exe of psftp.exe as well.

dd

todo:

* atlantic.net
* SSH: 
  * enable keys w/ passwords
  * disable passwords
  * Windows:
    * [putty](http://www.putty.org/)
