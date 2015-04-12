---
layout: post
title:  "Setting Up Your VPS"
date:   2015-04-12 06:39:00
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
has to go through your home connection. Since the VPN server in your
house is just a relay, it has to retransmit all data it receives, thus all
data to and from your laptop is limited to 1Mbps. Yikes!

Hosting the VPN on a woefully imbalanced connection might have 
made sense several years ago before 
VPS-hosting services were so plentiful. Now, you can rent a VPS for less than
it cost to maintain and power your own equipment. For example, as of 
April, 2015,
[Atlantic.net](https://www.atlantic.net) offers a Linux option with 256MB of
RAM, 1 virtual CPU, 10GB of storage, 1TB of outbound data, and unlimited
inbound data, all for $0.99 a month. While 256MB of RAM on a server is 
miniscule by today's standards, it is still enough to run a VPN with plenty
left over. Now the downside to a "Go Server", as they call it, is that it is
a flat rate for the month, unlike their larger servers which are billed only
when they are provisioned (i.e. up) on a per-second basis.

So, in today's post we'll go through setting and securing our baby server, 
leaving the details of actually setting up the VPN for a later post. The
following steps utilize the command line and are Linux-centric, but the process
is very similar for those on Windows systems using 
[Cygwin](http://www.cygwin.org/) or [Putty](http://www.putty.org/). 

### Generating Your SSH Key ###

The first thing to do, before actually creating our VPS, is to generate a SSH
authentication keys that we will use to log onto the machine. This is a 
significantly more secure method of logging into a remote machine than standard 
passwords - particularly if you protect your key with a passphrase. This is for
a couple of reasons: A) keys are significantly more difficult to crack with
brute-force due to their larger size and random composition, and B) keys allow
you to authenticate over a network without ever sending your password over
the network where an eavesdropper can intercept it. If you protect your
key with a passphrase, the passphrase is only handled on the local machine
and never traverses the network. A good write-up on SSH keys can be found
[here](https://wiki.archlinux.org/index.php/SSH_keys).

To generate our private/public key pair, use the following command:
{% highlight bash %}
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa.my_vpn
{% endhighlight %}
You will be prompted for an optional passphrase. If you specify a passphrase,
you will be prompted to re-enter this phrase every time you attempt to use the
key. If you want to be able to log in without specifying a passphrase, leave 
this blank.

The options used are:

* *-t rsa* - Specify that we want to create a RSA key pair.
* *-b 4096* - Specify that we want our key to be 4096 bits.
* *-f ~/.ssh/id_rsa.my_vpn* - Specify the path and base name of our key pair.

This will generate two files:

* *~/.ssh/id_rsa.my_vpn* - Your private key. Keep this secret and back it up!
* *~/.ssh/id_rsa.my_vpn.pub* - Your public key. Back it up! The contents of this
  file will get transferred to all machines you wish to use this key to log in
  to.

### Generate a New Root Password ###

Next, we want to take a moment to come up with a unique password to use as the
root password. Yes, I know we just generated a key so that we never have to 
actually enter the root password, but we still need to set one. So, pick a 
password that is reasonably secure. Optionally, you can generate a completely
random one and store it in a password manager, such as
[keepassx](https://www.keepassx.org/).

### Acquire Your VPS ###

Alright, now to actually acquire a VPS. Navigate to <https://www.atlantic.net>
and click "Create a Server", if you don't already have an account. If you 
already have an account, then log into your account, and click "Add Server" 
under the "Manage Servers" header on the left. You'll be prompted for the
following:

* Server Name - Choose whatever you want. For our purposes it is simply used to
  ID your server in the user panel.
* Location - Choose a geographical location closest to where you will be using
  this the most (i.e. city you live in).
* Select OS - I'll be referring to a Fedora install, but the principles should
  be the same for Ubuntu, Debian, or Centos. There is no real advantage to pick
  32bit over 64bit.
* Plan - Pick GO.
* Enable backups - If you want, but its cheaper to use rsync to backup your 
  VPS to your home machine.

After you hit continue you will be asked for your name and billing and contact
info. You know the drill. And yes, be honest. After you complete the account
creation and e-mail verification process, Atlantic.net will e-mail you the IP
address and password for your server. Once you receive this e-mail immediately
SSH into your box as root:
{% highlight bash %}
ssh root@<IP ADDRESS OF VPS>
{% endhighlight %}
Once logged in you will be asked to change the password. Enter the password
you came up with above.

### Installing Your SSH Key ###

Now it is time to install your SSH key so that you can use it to log in instead
of using the root password. To do so, on your home machine (where you generated
the keys), open a new terminal and type the following:
{% highlight bash %}
cat ~/.ssh/id_rsa.my_vpn.pub | ssh root@<IP ADDRESS OF VPS> 'cat >> ~/.ssh/authorized_keys'
{% endhighlight %}
You will be prompted for the root password again, go ahead and enter it.

In your terminal that is still logged into your VPS as root, type:
{% highlight bash %}
chmod 600 ~/.ssh/authorized_keys
{% endhighlight %}
This is necessary for newer versions of SSH that have an added safety measure 
that imposes strict rules on the file permissions of authorized_keys.

### Install Nano ###
By default, the only text editor installed is vi. 
For those that are not comfortable with vi, install the text editor nano:
{% highlight bash %}
yum install -y nano
{% endhighlight %}

From here on out, replace references to vi with nano.

### Disabling SSH Login Passwords ###

Lastly, we want to disable logging in with passwords in order to prevent
an attacker from brute-forcing the password. Absolutely *DO NOT* log out
until we verify that you can properly log in after completing this step.

To disable password logins via SSH, edit the SSH daemon config file. The
anxious will want to create a backup copy, first.
{% highlight bash %}
vi /etc/ssh/sshd_config
{% endhighlight %}

And look for the line that says:
{% highlight bash %}
PasswordAuthentication yes
{% endhighlight %}

Save and exit, then restart the SSH daemon:
{% highlight bash %}
systemctl restart sshd
{% endhighlight %}

Now try to log in with your certificate from your home machine:
{% highlight bash %}
ssh -i ~/.ssh/id_rsa.my_vpn root@<IP ADDRESS OF VPS>
{% endhighlight %}

If you were able to successfully log in, then your done. If not, go back and 
review installing your SSH key.

### Simplifying SSH Login ###

An optional step is to simply the SSH login command by setting up an identity
in the SSH user config file on your home machine. Using your favorite text 
editor, edit the file ~/.ssh/config (creating if necessary) and add the
following:
{% highlight bash %}
HOST vpn
     Hostname <IP ADDRESS OF VPS>
     IdentityFile ~/.ssh/id_rsa.my_vpn
     IdentitiesOnly yes
{% endhighlight %}

This allows you to log in to your machine with the following:
{% highlight bash %}
ssh root@vpn
{% endhighlight %}

### Fini! ###

Congratulations! You've now set up your own VPS and secured it against 
brute-force password attacks. Next time we'll get to the main event - setting
up our own VPN to protect our internet traffic from prying eyes.
