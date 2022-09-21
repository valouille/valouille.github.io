---
title: Voir le switch ainsi que le port sur lequel une machine est branchée avec tcpdump
author: VaLouille
type: post
date: 2013-04-18T15:40:03+00:00
url: /2013/04/voir-le-switch-ainsi-que-le-port-sur-lequel-une-machine-est-branchee-avec-tcpdump/
categories:
  - linux
  - sysadmin
tags:
  - cdp
  - cdpr
  - linux
  - tcpdump

---
Une alternative à cdpr consite à utiliser tcpdump pour lire les informations contenues dans les trames CDP. En effet, cdpr est gourmand en ressources et est assez lent. Voici ce que l&rsquo;on peut faire avec tcpdump :

```
tcpdump -nn -v -i eth0 -s 1500 -c 1 'ether[20:2] == 0x2000'
```

```
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 1500 bytes
15:37:08.888679 CDPv2, ttl: 180s, checksum: 692 (unverified), length 373
Device-ID (0x01), length: 7 bytes: 'sw02.th2'
Version String (0x05), length: 186 bytes: 
  Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 12.2(44)SE6, RELEASE SOFTWARE (fc1)
  Copyright (c) 1986-2009 by Cisco Systems, Inc.
  Compiled Mon 09-Mar-09 18:10 by gereddy
Platform (0x06), length: 22 bytes: 'cisco WS-C2960G-48TC-L'
Address (0x02), length: 13 bytes: IPv4 (1) 10.12.0.254
Port-ID (0x03), length: 19 bytes: 'GigabitEthernet0/20'
Capability (0x04), length: 4 bytes: (0x00000028): L2 Switch, IGMP snooping
Protocol-Hello option (0x08), length: 32 bytes: 
VTP Management Domain (0x09), length: 0 byte: ''
1 packets captured
10 packets received by filter
0 packets dropped by kernel
```
