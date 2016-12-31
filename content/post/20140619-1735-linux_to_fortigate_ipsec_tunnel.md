+++
title = "Establishing IPSec tunnel between Linux and Fortigate"
date = "2014-06-19T17:37:00+02:00"
tags = ["IPSec", "VPN", "Tunneling"]
+++


Here is an example of a tunnel setup beetween Linux and Fortigate. It uses StrongSwan.

    conn myconn
        type=tunnel
        left=[local_ip_address]
        leftsubnet=[local_subnet_to_be_accessed_through_tunnel]
        right=[remote_ip_address]
        rightsubnet=[remote_subnet_to_be_accessed_through_tunnel]
        keyexchange=ike
        ikelifetime=28800s
        ike=aes256-sha1-modp1536
        authby=secret
        keylife=1800s
        esp=aes256-sha1
        auto=start

This configures the forwarding policy. If you want to access the remote end of the tunnel from the gateway, set up a virtual interface and configure it with an address within the local_subnet.
