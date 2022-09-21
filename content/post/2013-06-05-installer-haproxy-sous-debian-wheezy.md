---
title: Installer HAproxy sous Debian Wheezy
author: VaLouille
type: post
date: 2013-06-05T14:32:23+00:00
url: /2013/06/installer-haproxy-sous-debian-wheezy/
categories:
  - linux
  - sysadmin
tags:
  - debian
  - haproxy
  - wheezy

---
HAproxy n&rsquo;est plus dans les depots Debian Wheezy, car l&rsquo;équipe n&rsquo;avait pas corrigé certains bugs. 

Il est cependant possible de l&rsquo;installer en ajoutant les backports. Pour cela, simplement ajouter la ligne suivante dans le fichier `/etc/apt/sources.list` :

```
deb http://ftp.debian.org/debian/ wheezy-backports main
```

Et :

```
apt-get update ; apt-get install haproxy
```
