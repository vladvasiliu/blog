+++
title = "Setting default NFSv4 ACLs on FreeBSD"
date = "2013-08-17T21:11:00+02:00"
tags = ["FreeBSD", "ZFS", "ACL", "sysadmin"]
+++

On FreeBSD with ZFS, only NFSv4 ACLs are supported. At the time of this writing, the [FreeBSD documentation regarding ACLs](https://wiki.freebsd.org/NFSv4_ACLs) is a bit vague. Specifically, it doesn’t talk about inheritable attributes.

The `-d` argument, used to define default POSIX ACLs is not supported. Instead, there are some flags one can set in order to define how ACLs are inherited. For an explanation, look in the manual for `setfacl`, section *ACL inheritance flags*.

Let’s say you want all the files and directories inside `somedir` to be readable by the users in the group `somegroup`. You would do the following:

```
setfacl -m g:somegroup:rx:fd:allow somedir
```

This sets `somedir` to be readable and executable by `somegroup` and those properties will be inherited by all new files (the `f` flag) and directories (`d` flag).

Note, however, that this only applies to newly created files. In the FreeBSD implementation of `setfacl` there is no recursive option. One way of applying the changes to the subtree is:

```
find . -type d -exec setfacl -m g:somegroup:rx:fd:allow {} \;
find . -type f -exec setfacl -m g:somegroup:r::allow {} \;
```

The first line sets the previous ACLs on all the subdirectories, while the second sets the ACL on all the files.

In order for this to work, some ZFS attributes must be set: `aclinherit=passthrough-x` and `aclmode=passthrough`.
See [here](http://docs.oracle.com/cd/E19082-01/817-2271/gbaaz/index.html) for the ZFS properties documentation.
