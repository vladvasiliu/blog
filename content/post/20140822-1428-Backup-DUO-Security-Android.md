+++
title = "How to transfer Duo Security installation to a new phone on Android"
date = "2014-08-22T14:31:00+02:00"
tags = ["Android", "Backup"]
description = ""
+++

The Duo Security application can be used to generate one time passwords for
online services, as an alternative to Google Authenticator.

However, it does not offer a way to backup the configuration, so if for some
reason it becomes unusable (say if your phone is stolen or broken) you will have
to reinitialise all the accounts.

There is however a way to get to this informaiton. It requires however the phone
to be "rooted". The configuration file is found in
`/data/data/com.duosecurity.duomobile/files/duokit/accounts.json`

If your new phone is not rooted (or is not an Android devide) you can get the
otp secrets from this file. In the application, when adding a new account,
 select "no barcode", and enter the otp secret in the `key` field.
