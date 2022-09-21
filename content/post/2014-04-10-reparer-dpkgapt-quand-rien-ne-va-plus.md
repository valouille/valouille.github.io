---
title: Réparer DPKG/APT quand rien ne va plus
author: VaLouille
type: post
date: 2014-04-10T14:02:31+00:00
url: /2014/04/reparer-dpkgapt-quand-rien-ne-va-plus/
categories:
  - linux
  - sysadmin
tags:
  - apt
  - debian
  - dpkg
  - libc6
  - libgcc1
  - multiarch-support

---
J&rsquo;ai récemment eu les erreurs suivantes :

```
dpkg: regarding .../libgcc1_1%3a4.7.2-5_amd64.deb containing libgcc1:amd64, pre-dependency problem:
 libgcc1 pre-depends on multiarch-support
  multiarch-support is unpacked, but has never been configured.

dpkg: error processing /var/cache/apt/archives/libgcc1\_1%3a4.7.2-5\_amd64.deb (&#8211;install):
   
pre-dependency problem &#8211; not installing libgcc1:amd64

Preparing to replace libc6:amd64 2.13-38+deb7u1 (using .../libc6_2.13-38+deb7u1_amd64.deb) ...
dpkg: error processing /var/cache/apt/archives_old/libc6_2.13-38+deb7u1_amd64.deb (--install):
 subprocess new pre-installation script returned error exit status 2
Setting up multiarch-support (2.13-38+deb7u1) ...
Setting up libgcc1:amd64 (1:4.7.2-5) ...
Errors were encountered while processing:
 /var/cache/apt/archives/libc6_2.13-38+deb7u1_amd64.deb
```

Cela venait du fait que dpkg était totalement cassé et ne savait plus quels paquets étaient installés. Il m&rsquo;affichait ceci :

```
# dpkg -l | grep libc6
iU  libc6:amd64        2.13-38+deb7u1  amd64        Embedded GNU C Library: Shared libraries
```

Normalement, `/var/lib/dpkg/status-old` et `/var/lib/dpkg/available-old` permettant de remettre de l&rsquo;ordre en les copiant :

```
cp /var/lib/dpkg/status-old /var/lib/dpkg/status
cp /var/lib/dpkg/available-old /var/lib/dpkg/available
```

Mais mes fichiers étaient erronés. Du coup, j&rsquo;ai pu m&rsquo;en sortir avec la commande suivante :

```
dpkg-deb -c /var/cache/apt/archives/libc6_2.13-38+deb7u1_amd64.deb | awk {'print $6'} | cut -f2- -d. | sed 's|^/$|/.|' | sed 's|/$||' > /var/lib/dpkg/info/libc6:amd64.list
apt-get upgrade -f
```
