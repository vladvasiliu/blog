+++
title = "MySQL Replication - I/O Error Code 1045"
date = "2013-08-17T21:05:00+02:00"
tags = ["MySQL"]
draft = false

+++

Setting up replication on MySQL may yield the following error:

```
[ERROR] Slave I/O: error connecting to master, Error code 1045
```

The MySQL troubleshooting page gives a few hints, talking about credentials for the replication user. However, attempting a connection via the command line client works, so the password is OK and the slave can reach the master. As it turns out, for replication purposes, the maximum password length is 32 characters.
