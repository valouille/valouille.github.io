---
title: Installer xdiff sous Debian avec PHP 5.4
author: VaLouille
type: post
date: 2013-05-06T09:54:55+00:00
url: /2013/05/installer-xdiff-sous-debian-avec-php-5-4/
categories:
  - linux
  - sysadmin
tags:
  - debian
  - php
  - xdiff

---
Voici un paquet permettant d&rsquo;installer l&rsquo;extension xdiff sous Debian :

Télécharger [php5-xdiff-5.4-1.5.2-1+awh_amd64.deb.tar.gz][1] et l&rsquo;installer :

```
wget https://blog.valouille.fr/wp-content/uploads/2013/05/php5-xdiff-5.4-1.5.2-1+awh_amd64.deb.tar.gz
tar xvzf php5-xdiff-5.4-1.5.2-1+awh_amd64.deb.tar.gz
dpkg -i php5-xdiff-5.4-1.5.2-1+awh_amd64.deb
```

On vérifie qu&rsquo;il est bien présent :

```
# php -r "phpinfo();" | grep xdiff
/etc/php5/cli/conf.d/20-xdiff.ini
xdiff
xdiff support => enabled
libxdiff version => LibXDiff v0.23 by Davide Libenzi <davide@xmailserver.org>
```

 [1]: https://blog.valouille.fr/wp-content/uploads/2013/05/php5-xdiff-5.4-1.5.2-1+awh_amd64.deb.tar.gz
