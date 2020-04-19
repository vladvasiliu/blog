+++
title = "Running rabbitmqctl as root"
date = "2014-04-25T11:42:00+02:00"
tags = ["RabbitMQ"]
description = ""
+++

On some Linux distributions, running `rabbitmqctl` as root may fail with the following error:

```
    Error: unable to connect to node rabbit@host: nodedown



    DIAGNOSTICS

    ===========



    attempted to contact: [rabbit@host]



    rabbit@host:

      * found rabbit (port 25672)

      * TCP connection succeeded

      * suggestion: hostname mismatch?

      * suggestion: is the cookie set correctly?



    current node details:

    - node name: rabbitmqctl19092@localhost

    - home dir: /root

    - cookie hash: B+onVMMDnR1uRT2nuD24VA==
```

Assuming that the hostname is correct and running `rabbitmqctl` as the rabbitmq user works, this is a cookie mismatch problem.

In order to be able to run `rabbitmqctl` as root, it must use the same cookie as the one used by the server. This cookie is usually found in `/var/lib/rabbitmq/.erlang.cookie`. To use this cookie, the `HOME` environment must be set to `/var/lib/rabbitmq`. Either create a wrapper around `/usr/bin/rabbitmqctl` that sets the correct environment, or edit this file to add `HOME=/var/lib/rabbitmq` on the line that starts with `exec ${ERL_DIR}`.
