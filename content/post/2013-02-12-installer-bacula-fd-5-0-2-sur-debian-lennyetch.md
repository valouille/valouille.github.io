---
title: Installer bacula-fd 5.0.2 sur Debian Lenny/Etch
author: VaLouille
type: post
date: 2013-02-12T15:18:09+00:00
url: /2013/02/installer-bacula-fd-5-0-2-sur-debian-lennyetch/
categories:
  - sysadmin
tags:
  - bacula
  - debian
  - etch
  - lenny

---
Voici des paquets Bacula 5 qui fonctionnent sous Debian 4.0. Avec les versions précédentes (< 5.0), certaines fonctionnalités de bacula ne sont pas disponibles. Si vous avez besoin d'utiliser bacula-fd 5 sur des vieilles version de debian, voici comment faire : 

  * Sous Lenny, il suffit de les installer via les backports :
Ajouter au fichier /etc/apt/sources.list les lignes suivantes :

```
deb http://ftp.debian.org/debian-backports lenny-backports main
deb-src http://ftp.debian.org/debian-backports lenny-backports main
```

```
apt-get update
apt-get install -t lenny-backports bacula-fd
```

Sous Etch, il faut utiliser les paquets suivants : [bacula_5.0.2-etch.tar.gz][1]

```
tar xvzf bacula_5.0.2-etch.tar.gz
```

Sur une machine AMD64 :

```
dpkg -i bacula/*amd64.deb
```

Sur une machine i386 :

```
dpkg -i bacula/*i386.deb
```

Il y a aussi la libpyton2.6 au besoin, si vous rencontrez des problèmes avec le .deb. Elle s&rsquo;utilisent comme ceci :

```
cp bacula/libpython2.6.so.1.0_amd64 /usr/lib/libpython2.6.so.1.0
```

ou

```
cp bacula/libpython2.6.so.1.0_i386 /usr/lib/libpython2.6.so.1.0
```

```
ln -s /usr/lib/libpython2.6.so.1.0 /usr/lib/libpython2.6.so.1
```

Puis il ne reste plus qu&rsquo;à tester :

```
/etc/init.d/bacula-fd start
```

 [1]: https://blog.valouille.fr/wp-content/uploads/2013/02/bacula_5.0.2-etch.tar.gz
