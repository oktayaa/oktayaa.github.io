---
layout: post
title: "it's virtual all the way down"
categories:
  - geeking out
tags:
  - linux
  - lxc
  - lxd
  - docker
  - kvm
  - qemu

published: true
---

>*Various virtualization and container technologies nest within each other very well and provide different levels of isolation. Here's one example from my server.*

The main machine is not virtual at all. It's a pretty beefy dedicated server or if we're being hip, `bare-metal`


```
[oktay@kvm ~]$ uname -a
Linux kvm.DOMAIN.com 3.10.0-1062.18.1.el7.x86_64 #1 SMP Tue Mar 17 23:49:17 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```


I am running `qemu-kvm` with `libvirt` on this machine.


```
[oktay@kvm ~]$ virsh list
 Id    Name                           State
----------------------------------------------------
 5     UbuntuServerOktay              running
 6     ubuntu16.04server              running
 7     ERPCentos7New            running
 8     virt01.debian8                 running
```

Let's pick one vm that has nested virtual stuffs in it.


```
[oktay@kvm ~]$ virsh console 5
Connected to domain UbuntuServerOktay
Escape character is ^]

xpufx login: oktay
Password:
Last login: Sat Mar 28 11:35:47 UTC 2020 on ttyS0
Welcome to Ubuntu 19.10 (GNU/Linux 5.3.0-42-generic x86_64)
```

This is a virtual machine running Ubuntu with lxd installed from `snap`. I am not really sold on snaps. To me they sound like installing software on OSX but it's how lxd is installed so I didn't fight it. (I do remove snapd on lxc images. Yes even the minimal cloud images have it.)

I have a bunch of lxc containers here. lxc containers are usually bigger than app containers, but they provide a lot more flexibility since they provide a whole OS environment.

```
oktay@xpufx:~$ lxc list -c n,4
+-------------+----------------------+
|    NAME     |         IPV4         |
+-------------+----------------------+
| alpine-edge | 10.50.0.250 (eth0)   |
+-------------+----------------------+
| dockers     | 172.17.0.1 (docker0) |
|             | 10.50.0.50 (eth0)    |
+-------------+----------------------+
| eaonmin     | 10.50.0.192 (eth0)   |
+-------------+----------------------+
| grafana     | 10.50.0.30 (eth0)    |
+-------------+----------------------+
| lamp        | 10.50.0.20 (eth0)    |
+-------------+----------------------+
| pihole      |                      |
+-------------+----------------------+
| rundeck     | 10.50.0.60 (eth0)    |
+-------------+----------------------+
oktay@xpufx:~$
```

Let's pick one and keep digging.

```
oktay@xpufx:~$ lxc exec dockers bash
root@dockers:~# docker container ls --format 'table {{.Names}}\t{{.Status}}'
NAMES               STATUS
telegraf            Up 13 hours
influxdb            Up 13 hours
grafana             Up 13 hours
privoxy             Up 13 hours
pihole              Up 14 hours (healthy)
portainer           Up 14 hours
```

I am in the process of moving some lxc containers into docker containers. That's why the same names such as portainer and grafana appear in multiple places.

This is a good time to mention when one should use lxc vs docker. The `grafana` instance in lxc actually also includes `influxdb` and `telegraf` installed with OS packages. In docker, I split them up. One app per container is suggested, and makes sense, although it is not enforced by docker.

Moving on. We're at the bottom of the virtualization staircase. Let's see what's going on in a pihole docker container, running in an Ubuntu LXC host, that's running in an Ubutu qemu VM, installed on a dedicated server running Centos 7.
```

root@dockers:~# docker exec -it pihole bash
root@1215803f4731:/# ps -axw
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:00 s6-svscan -t0 /var/run/s6/services
   28 ?        S      0:00 s6-supervise s6-fdholderd
 1282 ?        S      0:00 s6-supervise lighttpd
 1284 ?        S      0:00 s6-supervise cron
 1285 ?        S      0:00 s6-supervise pihole-FTL
 1287 ?        Ss     0:00 bash ./run
 1289 ?        Ss     0:00 bash ./run
 1299 ?        S      0:09 lighttpd -D -f /etc/lighttpd/lighttpd.conf
 1303 ?        S      0:00 /usr/sbin/cron -f
 1325 ?        Ss     0:00 /usr/bin/php-cgi
 1326 ?        S      0:02 /usr/bin/php-cgi
 1327 ?        S      0:02 /usr/bin/php-cgi
 1328 ?        S      0:02 /usr/bin/php-cgi
 1329 ?        S      0:02 /usr/bin/php-cgi
 2061 ?        Ss     0:00 bash ./run
 2065 ?        Sl     0:59 pihole-FTL no-daemon
 5565 pts/0    Ss+    0:00 bash
26529 pts/1    Ss     0:00 bash
26604 pts/1    R+     0:00 ps -axw
root@1215803f4731:/#
```

I am personally happy that pihole's <em>selection of things that I wouldn't have exactly done that way</em> are contained as deep down as possible.
