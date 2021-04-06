---
layout: post
title: "Two simultaneous wireguard connections on windows"
categories:
  - geeking out
tags:
  - windows
  - wireguard
  - tunsafe
  - vpn
published: true
---

>
The current `wireguard` client on Windows supports only one connection at a time. You will notice that the title is **two** connections, not **multiple**, because this is just a workaround to add just one more connection.

-- This is now obsolete since wireguard added this functionality. Info coming `soon`

This will be brief. Basically wireguard currently only allows one connection on Windows. However there's another gui for wireguard on windows which can be used without disconnecting wireguard. This is possible because each uses its own tun adapter.

So the instructions are to use the official wireguard connection for one of the tunnels and the `tunsafe` client for the other one. I believe both clients will have multiple connections on the gui in the near future.

Note: wireguard people don't seem to like tunsafe people. So don't be surprised If you've never heard about tunsafe before. It's an open source client just like wireguard's and uses the same wireguard server software. In other words, they just make a client. As a bonus, their client comes with a preconfigured connection for their vpn service which is good for testing or to use in a pinch.

Addendum: The tunsafe changelog claims multiple connections since October of 2018.

`6.The 'tunsafe' command line tool supports multiple wireguard
  sessions simultaneously using different tun interfaces.`
