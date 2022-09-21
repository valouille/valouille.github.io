---
title: Compiler un module PECL en paquet Debian
author: VaLouille
type: post
date: 2014-10-17T08:48:55+00:00
url: /2014/10/compiler-un-module-pecl-en-paquet-debian/
categories:
  - linux
  - sysadmin
tags:
  - apt
  - debian
  - package
  - pecl
  - php

---
Voici la procédure pour générer un .deb à partir d’un .tar.gz PECL de la façon la plus simple qui soit.

On aura besoin d’utiliser dh-make-pecl pour générer le répertoire debian :

```
apt-get install dh-make-php php5-dev ubuntu-dev-tools build-essential
```

On choisit le nom et l&rsquo;adresse mail du mainteneur du paquet

```
export DEBFULLNAME="Valerian Beaudoin"
export DEBEMAIL="mail@domaine.com"
```

On télécharge l&rsquo;extension PECL qui nous intéresse :

```
wget http://pecl.php.net/get/apcu-4.0.7.tgz
```

Puis on utilise dh-make-pecl qui va générer les fichiers debian/control, debian/rules … :

```
dh-make-pecl apcu-4.0.7.tgz
```

On se déplace dans le répertoire créé :

```
cd php-apcu-4.0.7/
```

On lance la compilation :

```
debuild
```

Le paquet `../php5-apcu\_4.0.7-1\_amd64.deb` est maintenant créé !
