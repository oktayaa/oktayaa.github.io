---
layout: post
title: How to change LXC container IP
categories:
  - geeking out
tags:
  - linux
  - lxc
  - lxd
published: false
---

>*If you haven't specified an IP while creating an `LXC` instance, you might end up with an IP that doesn't suit your needs. There's a quick and easy way of setting IP addresses for LXC containers. It is faster and less requires less tinkgering than trying to fix the IP either on the host dhcp side or the guest dhclient side.*

In the below example we're changing the IP address for the container called `grafana`. We'll have to stop the container before we can do it. The IP we're setting for this example is `10.0.0.30`. The bridge interface on the host is `lxdbr0` and the interface name inside the container is `eth0`. Change these parameters for your environment.



{: .box-note}
<pre>
lxc stop grafana
lxc network attach lxdbr0 grafana eth0 eth0
lxc config device set grafana eth0 ipv4.address 10.0.0.30
lxc start grafana
</pre>

You can check the change using `lxc list`

In my case the IPs come from `dnsmasq` on the host. I believe this is the default setting. The above method should work for static IPs (without DHCP) as well.
