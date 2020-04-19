+++
title = "IPv6 prefix delegation on FreeBSD router"
date = "2018-08-27T09:22:00+02:00"
tags = ["sys", "bsd", "freebsd", "ipv6", "dhcp", "dhcpv6", "pd", "prefix delegation", "sfr", "routing", "SLAAC"]
description = ""
+++

# Goal
I'm attempting to set up a router box in order to replace my SFR router on a home network.
As my SFR plan has native IPv6, this box will request a prefix delegation and set up IPv6 routing for the computers on the home network.
It will also handle NAT for IPv4.

This is supposed to be a drop-in replacement for the SFR provided box, so it will have to:

* Get an IPv4 via DHCP
* Get an IPv6 PD via DHCPv6
* Announce itself as an IPv6 router to the internal network via SLAAC

# How this works

* The router box will ask for a prefix from the provider. This is usually a /56 or larger.
* The router configures an IP from a subprefix of the provided prefix on the internal interfaces.
* The router then advertises this prefix as well as itself to clients over the internal interface.

# Prerequisites

This is based on the following:

* A box with two or more network interfaces
* FreeBSD 11.2 (tested on FreeBSD 11.2-release-p2)
* KAME DHCP6 (tested on dhcp6-20080615.2)
* rtadvd (from base installation)

# Configuration

The following examples are based on the following convention:

* `re0` is the wan interface, going to my ISP.
* `re1` is the lan interface, going to my internal network.
* `bridge0` is an internal bridge for hosting VMs that run on the router.

## System

Set the following sysctls in `/etc/sysctl.conf`:

```rc.conf
net.inet6.ip6.rfc6204w3=1
```

This is not strictly necessary, as it is handled by `ipv6_cpe_wanif` in `rc.conf`.

Set up interfaces and enable routing in `rc.conf` ([see manual][rc.conf manual]):

```rc.conf
ipv6_cpe_wanif="re0"
ipv6_activate_all_interfaces="YES"
ipv6_gateway_enable="YES"

ifconfig_re0_ipv6="inet6 accept_rtadv up"
ifconfig_re1_ipv6="inet6 -accept_rtadv up"

rtadvd_enable="YES"
rtadvd_interfaces="re1 bridge0"

dhcp6c_enable="YES"
dhcp6c_interfaces="re0"

```

## DHCPv6 client

This is required to request a prefix delegation from the ISP. Note that this won't setup a routable address on the wan interface. As this is supposed to be a router, it's not required. IPv6 communication to the router is done via link local addresses.

Install KAME/WIDE dhcp6c client with pkg (or from _net/dhcp6_ in ports):

```
pkg install dhcp6
```

Add to `/usr/local/etc/dhcp6c.conf` ([see manual][dhcp6c.conf manual]):

```rc.conf
# Request a prefix delegation on re0
interface re0 {
        send    ia-na 1;
        send    ia-pd 1;
        send    rapid-commit;
};

# Handle the response
id-assoc pd 1 {
        # Assign a /64 per address (rtadvd may choke with other prefix lengths)
        prefix ::/64 1800;

        # Assign a /64 to re1 with id 0.
        prefix-interface re1 {
                sla-id 0;
                sla-len 8;
        };

        # Assign a /64 to bridge0 with id 1.
        prefix-interface bridge0 {
                sla-id 1;
                sla-len 8;
        };
};

id-assoc na 1 {
};
```

My ISP delegates me a /56. If I want to obtain a /64, I have to cut 8 bits from it. This is what the `sla-len 8` statement does.

What basically happens is it builds an IPv6 of the following form: `prefix:sla:local_addr`. This is then configured on the interface corresponding to the `prefix-interface` statement. This is on the client side.

The _sla_ becomes part of the prefix as advertised to the client subnets.

## rtadvd

This is a daemon that works on the client side of the router and handles host autoconfiguration via SLAAC:

* It announces the prefix of the interface to the hosts connected to that interface by periodically sending router advertisements and responding to router solicitations.
* It announces itself as the router for the segment by replying to neighbour solicitations and sending gratuitious neighbour advertisments.

rtadvd is part of the FreeBSD base system. It expects its configuration in `/etc/rtadvd.conf` ([see manual][rtadvd.conf manual]):

```rc.conf
default:\
       :prefixlen#64:\
```

If the prefixes on the client side are dynamic, as is the case when handled via dhcp6c, there is no need to specify them in the configuration file. rtadvd will use the ones configured on the interface and handle prefix changes.

If you need different settings per interface, you can specify the interface name instead of _default_ in the config file.


## Firewall

If you're using PF, be sure to only NAT IPv4 packets. The below configuration shows what's required to allow IPv6 autoconfiguration.

```pf.conf
wan = "re0"
lan = "re1"

nat log on $wan inet from ! $wan to any -> ($wan)

# Allow DHCP6
pass in log on $wan inet6 proto udp from any to ( $wan ) port dhcpv6-client keep state
pass out log on $wan inet6 proto udp from self to any port dhcpv6-server keep state

pass log inet6 proto icmp6 all icmp6-type {neighbradv, neighbrsol, routersol, routeradv}
```

As dhcp6c doesn't get a route, it's necessary that the router be able to send router solicitations and receive advertisments on the wan interface (see entry for `ipv6_cpe_wanif` in [rc.conf manual][]).

Add other firewall rules as necessary. The above rules won't allow ping for example.

# References

Resources that have helped me along the way:

## Manual pages

* [rc.conf manual][]
* [rtadvd.conf manual][]
* [dhcp6c.conf manual][]

## Forums and blogs

* [Setting up FreeBSD with Comcast IPv6](https://blog.crashed.org/setting-up-freebsd-with-comcast-ipv6/)
* [FreeBSD Forums - IPv6 Gateway](https://forums.freebsd.org/threads/ipv6-gateway.53522/)
* [Providing IPv6 DNS resolver data with radvd](https://www.systemajik.com/providing-ipv6-dns-resolver-data-using-radvd)




[rc.conf manual]: https://www.freebsd.org/cgi/man.cgi?query=rc.conf&apropos=0&sektion=0&manpath=FreeBSD+11.2-RELEASE+and+Ports&arch=default&format=html "rc.conf"
[rtadvd.conf manual]: https://www.freebsd.org/cgi/man.cgi?query=rtadvd.conf&apropos=0&sektion=0&manpath=FreeBSD+11.2-RELEASE+and+Ports&arch=default&format=html "rtadvd.conf"
[dhcp6c.conf manual]: https://www.freebsd.org/cgi/man.cgi?query=dhcp6c.conf&apropos=0&sektion=0&manpath=FreeBSD+11.2-RELEASE+and+Ports&arch=default&format=html "dhcp6c.conf"
