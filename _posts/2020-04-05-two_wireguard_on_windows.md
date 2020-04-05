---
layout: post
title: "two simultaneous wireguard connections on windows"
categories:
  - geeking out
tags:
  - windows
  - wireguard
  - tunsafe
  - vpn
published: true
---

>*Current `wireguard` client on Windows supports only one connection at a time. You will notice that the title is **two** connections, not **multiple**, because this is just a workaround to add just one more connection. *


This will be brief. Basically wireguard allows only one connection. However there's another gui for wireguard on windows which can be used without disconnecting wireguard. This is possible because each uses its own tun adapter.

So the instructions are to use the official wireguard connection for one of the tunnels and the tunsafe client for the other one. I believe both clients will have multiple connections on the gui in the near future.

Note: wireguard people don't seem to like tunsafe people. If you've never heard about tunsafe, this is probably why. However it's an open source client just like wireguard's and uses the same wireguard server software. In other words, they just make a client. As a bonus, their client comes with a preconfigured connection for their vpn service which is good for testing or to use in a pinch.
