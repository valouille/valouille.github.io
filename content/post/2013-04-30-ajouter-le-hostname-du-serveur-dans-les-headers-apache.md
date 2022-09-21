---
title: Ajouter le hostname du serveur dans les headers Apache
author: VaLouille
type: post
date: 2013-04-30T14:53:34+00:00
url: /2013/04/ajouter-le-hostname-du-serveur-dans-les-headers-apache/
categories:
  - linux
  - sysadmin
tags:
  - apache
  - debian
  - headers

---
Cela peut être pratique dans le cas où on utilise un cluster de serveur Apache de voir quel est le nom du serveur qui a servi la requête, pour du debug par exemple. Voici la procédure :

Tout d&rsquo;abord il faut activer le module headers :

```
a2enmod headers
```

Puis on ajoute la ligne suivante au fichier /usr/sbin/apache2ctl afin de générer une variable d&rsquo;environnement :

```
export HOSTNAME=`hostname`
```

Enfin, il faut ajouter les deux lignes suivantes à la fin du fichier /etc/apache2/apache2.conf :

```
PassEnv HOSTNAME
Header add X-Server-Name "%{HOSTNAME}e"
```

On restart apache :

```
/etc/init.d/apache2 restart
```

Et voilà, dans les headers on devrait avoir un ligne du genre :

```
X-Server-Name: hillys
```
