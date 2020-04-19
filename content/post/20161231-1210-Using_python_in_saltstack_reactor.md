+++
title = "Using Python in SaltStack reactor"
date = "2016-12-31T12:11:00+02:00"
tags = ["Sys", "salt"]
description = ""
+++

Sometimes you need to do some complex actions in a Salt Reactor in which case Python comes in handy.

The documentation is not very clear on how to write Python states. The main point is that Salt expects a structure that can be compiled to a Python dictionary. The run() function has to return a dictionary which resembles the state you would write with YAML.

Here's an example to add a domain to route 53:

```python
#!py

def run():
   '''Add domain name to route53'''

   full_name = data['name']

   subzone, name = full_name.split('.')
   zone = "some.zone.tld"

   try:
       aws_query = __salt__['cloud.full_query']()['aws']['ec2'][full_name]['dnsName']
   except KeyError:
       dns_setup_hsd = {'setup-dns':
           {'local.test.nop': [
               {'tgt': 'some.minion.id'},
               ]}}
   else:
       dns_setup_hsd = {'setup-dns':
           {'local.boto_route53.add_record': [
               {'tgt': 'some.minion.id'},
               {'arg': [
                   '.'.join((name, zone)),
                   aws_query,
                   zone,
                   'CNAME']}]}}

   return dns_setup_hsd

```

