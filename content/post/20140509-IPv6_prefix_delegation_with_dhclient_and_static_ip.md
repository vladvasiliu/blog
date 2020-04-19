+++
title = "IPv6 prefix delegation with static address and dhclient on Ubuntu / Debian"
date = "2014-05-09T10:47:00+02:00"
tags = ["IPv6", "DHCPv6"]
description = ""
+++

Some hosting providers, like [Online](http://online.net) in France provide IPv6 access via prefix delegation. You get a DUID that has to be sent to their DHCP server. The IP address is configured by the client.

The recommended method is using Dibbler, but this solution is pretty buggy.

On Ubuntu or Debian, you can use the provided dhclient to obtain the prefix delegation.

Below are samples of `dhclient6.conf` and `interfaces`. Note that in this configuration everything is configured by the client, but you could request a name server from the DHCP server.

`/etc/dhcp/dhclient6.conf`:

    interface "em1" {
        send dhcp6.client-id your:duid;
        request;
    }

`/etc/network/interfaces`:

    auto em1
    iface em1 inet6 static
        address your:ip:v6:address
        netmask your_nemask
        accept_ra 1
        pre-up /sbin/dhclient -cf /etc/dhcp/dhclient6.conf -6 -P em1
