+++
title = "Zone transfer setup for BIND with split horizon on the same zone"
date = "2014-12-08T10:09:00+02:00"
tags = ["DNS", "BIND"]
+++

With the default configuration, a zone transfer from a master to a slave of BIND with split horizon has the slave transfer only the view of that zone that is available to its IP address. The documentation is a bit vague on the subject, but it turns out it is possible for the slave to transfer the zone definitions corresponding to all (or a subset) of the views.

The idea is to identify the slave for zone transfer purposes using something else that its IP address.

The way to make this work is by using keys to identify the slaves to the server, one key per view.

For example, key definitions:

    key "key1" {
      algorithm hmac-sha512;
      secret "[some_secret_1]";
    };

    key "key2" {
      algorithm hmac-sha512;
      secret "[some_secret_2]";
    };

ACLs:

    acl acl1 {
      key "key1";
      !key "key2";
      !key "key3";
      10.1.0.0/24;
    };

    acl acl2 {
      !key "key1";
      key "key2";
      !key "key3";
      10.2.0.0/24;
    };

The views:

    view "view1" {
      match-clients { acl1; };
      empty-zones-enable no;

      server 10.1.1.2 {
        keys "key1";
      };

      server 10.1.1.3 {
        keys "key1";
      };

      zone "." IN {
        type hint;
        file "/etc/namedb/named.root";
      };

      zone "my.zone.com" {
        type master;
        file "/etc/namedb/master/my.zone.com";
        allow-transfer {
          key "key1";
        };
        also-notify { 10.1.1.2; 10.1.1.3; };
      };

      allow-query { acl1; };
      allow-query-cache { acl1; };
      allow-recursion { acl1; };
    };

    view "view2" {
      match-clients { acl2; };
      empty-zones-enable no;

      server 10.1.1.2 {
        keys "key2";
      };

      server 10.1.1.3 {
        keys "key2";
      };

      zone "." IN {
        type hint;
        file "/etc/namedb/named.root";
      };

      zone "my.zone.com" {
        type master;
        file "/etc/namedb/master/my.zone.com";
        allow-transfer {
          key "key2";
        };
        also-notify { 10.1.1.2; 10.1.1.3; };
      };

      allow-query { acl2; };
      allow-query-cache { acl2; };
      allow-recursion { acl2; };

    };

The idea here is to identify the server by its key, hence it's important that in the ACL definition the keys are before the IP, or else the IP will always match the same ACL. Also, the servers must of course use a different key for each zone. The keys may be shared among servers, or every server can have it's own set of keys. Everybody else is identified by their IP address, and served the corresponding view.
