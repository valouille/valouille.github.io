---
title: Paquets Debian pour roundcube 0.9.0
author: VaLouille
type: post
date: 2013-04-19T16:14:31+00:00
url: /2013/04/paquets-debian-pour-roundcube-0-9-0/
categories:
  - Uncategorized
tags:
  - 0.9
  - 0.9.0
  - debian
  - package
  - packages
  - paquet
  - paquets
  - roundcube

---
Voici des paquets debian permettant d&rsquo;installer la dernière version de roundcube :
  
[roundcube_0.9.0.tar.gz][1]

Pour l&rsquo;installer :

```
wget https://blog.valouille.fr/wp-content/uploads/2013/04/roundcube_0.9.0.tar.gz
tar xvzf roundcube_0.9.0.tar.gz
dpkg -i roundcube_0.9.0-1_all.deb roundcube-core_0.9.0-1_all.deb roundcube-mysql_0.9.0-1_all.deb roundcube-plugins_0.9.0-1_all.deb
```

Ou pour utiliser postgreSQL à la place de mySQL:

```
dpkg -i roundcube_0.9.0-1_all.deb roundcube-core_0.9.0-1_all.deb roundcube-pgsql_0.9.0-1_all.deb roundcube-plugins_0.9.0-1_all.deb
```

 [1]: https://blog.valouille.fr/wp-content/uploads/2013/04/roundcube_0.9.0.tar.gz
