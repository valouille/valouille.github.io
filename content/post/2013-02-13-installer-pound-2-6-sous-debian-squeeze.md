---
title: Installer Pound 2.6 sous Debian Squeeze
author: VaLouille
type: post
date: 2013-02-13T11:13:23+00:00
url: /2013/02/installer-pound-2-6-sous-debian-squeeze/
categories:
  - sysadmin
tags:
  - debian
  - proxy

---
Pound est un reverse proxy open source et peut aussi être utilisé à des fins de loadbalancing. Voici la version 2.6 compatible avec Debian Squeeze compilée sous forme de paquets .deb. A télécharger ici : [pound-2.6-2-squeeze.tar.gz][1]

Pour l&rsquo;installer :

```
mkdir pound &amp;&amp; cd pound
wget https://blog.valouille.fr/wp-content/uploads/2013/02/pound-2.6-2-squeeze.tar.gz
tar xvzf pound-2.6-2-squeeze.tar.gz
dpkg -i *.deb
```

Attention, il est nécessaire d&rsquo;installer la libssl1.0.0.

 [1]: https://blog.valouille.fr/wp-content/uploads/2013/02/pound-2.6-2-squeeze.tar.gz
