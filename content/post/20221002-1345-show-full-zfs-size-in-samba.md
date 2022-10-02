---
title: "Show full ZFS size in Samba"
date: 2022-10-02T13:46:52+02:00
lastmod: 2022-10-02T13:46:52+02:00
draft: false
keywords: [zfs samba]
description: "How to coax Samba into showing the full size of a ZFS pool as the total share size"
tags: [zfs, samba]
categories: []
author: "Vlad Vasiliu"

toc: false
postMetaInFooter: true
hiddenFromHomePage: false

---

The default for Samba is to show the total space of a ZFS-backed share as the sum of available + used by the that dataset. When multiple datasets are present, the total size is confusing.

<!--more-->

## Goal

I want the "total share size" of the Samba share to show the total usable size of the underlying ZFS dataset.

## Environment

This is tested with Samba 4 and OpenZFS 2.1.5 on Linux with a single Samba share. It should probably work the same on other systems.


## Quick steps

Create a script for computing the total and avaiable size at the root of the pool. It should be owned by root and executable.

````bash
#!/bin/bash
read -r used available <<< "$(zfs list -Hp -o used,available fspool)"
total=$(($used + $available))
echo $total $available 1
````

Reference this script in your Samba share config.

````
[someshare]
   comment = Some share
   path = /home/dude/share
   valid users = user1
   public = no
   writable = yes
   printable = no
   dfree command = /usr/local/sbin/dfree.sh
````

## Explanation

By default, Samba will compute the total size of a share by summing the available and used size of the dataset hosting the share.
If there are other datasets taking up space, the total space reported will be wrong, and go down as the other datasets take up more space.

The `dfree` parameter allows to run a specific script that will return the total and available size of the share.

## References

* [dfree command](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html#idm2868)
