---
title: "Cloud-init doesn't append the SSH-key/password at first boot of an instance"
date: 2018-07-05T13:37:00+01:00
draft: false
author: VaLouille
type: post
categories:
  - OpenStack
tags:
  - OpenStack
  - instance
  - ssh
  - key
  - password
  - hostname
  - cloud-init
  - firewall
  - egress
  - ubuntu
  - 18.04
  - bionic
---

If cloud-init fails to append the specified SSH-key or password at the first boot of an OpenStack instance, or changing the hostname, it could just be a simple misconfiguration of a `security group`. Indeed, in the default `security group` or within any newly created `security group` the following rule is added by default :

| Direction    | Ether Type  | IP Protocol  | Port Range |   Remote IP Prefix | Remote Security Group | Local IP Prefix |
| :----------- | :---------- |:------------ | :--------- | :----------------- | :-------------------- | :-------------- |
| Egress       | IPv4        | Any          | Any        | `0.0.0.0/0`        | -                     | -               |

This rule allows all outgoing connections. Is this rule is deleted, cloud-init no longer succeed with every task. Indeed, cloud-init *does* an outgoing TCP connection to `http://169.254.169.254` to obtain metadatas from OpenStack. Is the default outgoing rule has been altered of deleted, this TCP connection is no longer allowed.

So, the following rule should be added in order to allow cloud-init to work :

| Direction    | Ether Type  | IP Protocol  | Port Range |   Remote IP Prefix   | Remote Security Group | Local IP Prefix |
| :----------- | :---------- |:------------ | :--------- | :-----------------   | :-------------------- | :-------------- |
| Egress       | IPv4        | TCP          | 80         | `169.254.169.254/32` | -                     | -               |


> Another reason specific to VIO (VMware Integrated Openstack) could be that cloud-init doesn't realize it's running on an OpenStack environment since the hypervisor itself is actually ESXi. To explicitely specifiy to cloud-init that it's working on OpenStack, the file `/etc/cloud/cloud.cfg.d/90_dpkg.cfg` is to be updated in the base image with the following content: 
```
datasource_list: [ OpenStack ]
```
Since only one datasource is specified, cloud-init knows *de facto* that it's running on OpenStack.

