---
title: Créer un paquet Debian vide
author: VaLouille
type: post
date: 2013-10-14T09:49:31+00:00
url: /2013/10/creer-un-paquet-debian-vide/
categories:
  - linux
  - sysadmin
tags:
  - debian
  - dummy
  - equivs
  - package

---
Pour créer un « dummy package », c&rsquo;est à dire un paquet Debian vide qui peut être utile pour satisfaire des dépendances, on utilise equivs. Par exemple, pour libkrb53

```
apt-get install equivs
equivs-control libkrb53
```

Cela va créer un fichier libkrb53. On l&rsquo;édite :

```
Section: misc
Priority: optional
Standards-Version: 2.9.2

Package: libkrb53
Version: 1.9.1
Maintainer: Valerian Beaudoin
Architecture: all
Description: Dummy libkrb53 package
```

Puis on le construit et on l&rsquo;installe

```
equivs-build libkrb53
dpkg -i libkrb53_1.9.1_all.deb
```
