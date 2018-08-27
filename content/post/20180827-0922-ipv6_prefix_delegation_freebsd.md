+++
title = "IPv6 prefix delegation on FreeBSD router"
date = "2018-08-27T09:22:00+02:00"
tags = ["sys", "bsd", "freebsd", "ipv6", "dhcp", "dhcpv6", "pd", "prefix delegation", "sfr", "routing", "SLAAC"]
+++

# Goal
I'm attempting to set up a router box in to replace my SFR router on a home network. As my SFR plan has native IPv6, this box will request a prefix delegation and set up IPv6 routing for the computers on the home network. It will also handle NAT for IPv4.

This is supposed to be a drop-in replacement for the SFR provided box, so it will have to:

* Get an IPv4 via DHCP
* Get an IPv6 PD via DHCPv6
* Announce itself as an IPv6 router to the internal network via SLAAC

# Prerequisites

# Configuration

## Network

## Firewall

# References
Resources that have helped me along the way:

* [Setting up FreeBSD with Comcast IPv6](https://blog.crashed.org/setting-up-freebsd-with-comcast-ipv6/)
* [FreeBSD Forums - IPv6 Gateway](https://forums.freebsd.org/threads/ipv6-gateway.53522/)
* [Providing IPv6 DNS resolver data with radvd](https://www.systemajik.com/providing-ipv6-dns-resolver-data-using-radvd)