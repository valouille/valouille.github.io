---
title: Résoudre les problèmes Grub liés à Grsec
author: VaLouille
type: post
date: 2013-04-30T15:42:30+00:00
url: /2013/04/resoudre-les-problemes-grub-lies-a-grsec/
categories:
  - linux
  - sysadmin
tags:
  - grsec
  - grsecurity
  - grub
  - grub2

---
Lorsque l&rsquo;on utilise un noyau Grsec et que l&rsquo;on souhaite faire un « update-grub », les erreurs suivantes peuvent se manifester :

```
# update-grub
Killed
Killed
Generating grub.cfg ...
Killed
```

Cela se matérialise comme ceci dans les logs kernel :

```
PAX: From 10.0.87.1: execution attempt in: <anonymous mapping>, 3c2cfe2a000-3c2cfe40000 3fffffe9000
PAX: terminating task: /usr/sbin/grub-probe(grub-probe):30554, uid/euid: 0/0, PC: 000003c2cfe3eca8, SP: 000003c2cfe3ec58
PAX: bytes at PC: 41 bb 70 40 40 00 49 ba a0 ec e3 cf c2 03 00 00 49 ff e3 00
PAX: bytes at SP-8: 0000000000000006 0000000000401d57 0000000000000000 0000000000622aa0 000003c2cfe3eca8 0000000000404196 0000000000000000 000003c2cfe3f025 000003c2cfe3ee18 0000000000403edd 000000000041a070
grsec: From 10.0.87.1: denied resource overstep by requesting 4096 for RLIMIT_CORE against limit 0 for /usr/sbin/grub-probe[grub-probe:30554] uid/euid:0/0 gid/egid:0/0, parent /usr/sbin/update-grub[update-grub:30537] uid/euid:0/0 gid/egid:0/0
```

La solution consiste à installer paxctl (ou chpax en fonction de la version de debian utilisée) :

```
apt-get install paxctl
```

Ensuite, on fait un backup des fichiers que l&rsquo;on va modifier :

```
cp -a /usr/sbin/grub-probe /usr/sbin/grub-probe.old 
cp -a /usr/sbin/grub-mkdevicemap /usr/sbin/grub-mkdevicemap.old 
cp -a /usr/sbin/grub-setup /usr/sbin/grub-setup.old
```

Enfin, en change les attributs :

```
paxctl -Cpemrxs /usr/sbin/grub-probe 
paxctl -Cpemrxs /usr/sbin/grub-mkdevicemap 
paxctl -Cpemrxs /usr/sbin/grub-setup
```

Il ne devrait plus y avoir de problème.
