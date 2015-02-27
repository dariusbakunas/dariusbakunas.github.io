---
layout: post
comments: true
title:  "Configure virtual networking in Xen"
date:   2015-02-27 9:40:00
categories:
- xen
- virtualization
permalink: configure-virtual-networking-in-xen
description: Creating virtual switch in Xen host (ie. internal or host-only network)
---


I was not able to find many guides on how to setup virtual networking inside Xen hypervisor. Comming from VirtualBox, I wanted to implement internal or host-only networking between my virtual machines.


## Setup

I'm currently using Xen 4.1.4 (with xl toolstack) on Debian Wheezy 64bit.

* Install vde2 package:

{% highlight bash %}
% sudo apt-get install vde2
{% endhighlight %}


* Edit /etc/network/interfaces to add virtual bridge adapter:

	Add these lines:

{% highlight bash %}  
auto tap0  
iface tap0 inet manual  
	vde2-switch -t tap0

auto xenbr1	
iface xenbr1 inet static
	bridge_ports tap0
	address 10.0.0.1
	netmask 255.255.255.0
{% endhighlight %}

	I've already had xenbr0 configured, so I chose to name it 'xenbr1'.
	If you don't want Xen host to be in this network, just remove static configuration from xenbr1 and change it to:

{% highlight bash %}
iface xenbr1 inet manual
{% endhighlight %}

* Restart networking:

{% highlight bash %}
% sudo /etc/inet.d/networking restart
{% endhighlight %}

* Or reboot:

{% highlight bash %}
% sudo reboot
{% endhighlight %}


* Now add this new adapter to your vm configuration files as usual:

{% highlight bash %}  
vif=['bridge=xenbr1']
{% endhighlight %}

	Do same for multiple virtual machines and they will be able to communicate using this virtual network. 
	Ofcourse, don't forget to set ip configuration for those devices inside virtual machines.