+++
title = "IPv6 routing and router advertisments on Ubuntu / Debian"
date = "2014-05-09T10:47:00+02:00"
tags = ["IPv6", "DHCPv6"]
description = ""
+++

If your Linux box gets its IPv6 routing information via router advertisments, you may have noticed that upon enabling forwarding, it disregards those advertisments and it loses the configured gateways.

The have this configuration working again, set `accept_ra 2` in `/etc/network/interfaces`. This value is not documented in the `interfaces` manual, but since kernel version 2.6.37 it allows the interface to both do forwarding and accept router advertisments.

[Source](http://strugglers.net/~andy/blog/2011/09/04/linux-ipv6-router-advertisements-and-forwarding/)
