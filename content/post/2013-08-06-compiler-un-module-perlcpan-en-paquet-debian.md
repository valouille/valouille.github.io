---
title: Compiler un module Perl/CPAN en paquet Debian
author: VaLouille
type: post
date: 2013-08-06T09:42:27+00:00
url: /2013/08/compiler-un-module-perlcpan-en-paquet-debian/
categories:
  - linux
  - sysadmin
tags:
  - compiler
  - cpan
  - debian
  - paquet
  - perl

---
Voici la procédure pour générer un .deb à partir d&rsquo;un .tar.gz de la façon la plus simple qui soit.

On aura besoin d&rsquo;utiliser dh-make-perl pour générer le repertoire debian, ainsi que d&rsquo;apt-file :

```
apt-get install dh-make-perl apt-file devscripts build-essential
```

On met à jour apt-file :

```
apt-file update
```

Ensuite, on télécharge le module que l&rsquo;on souhaite installer :

```
wget http://search.cpan.org/CPAN/authors/id/T/TO/TOKUHIROM/FCGI-Client-0.08.tar.gz
```

On l&rsquo;extrait :

```
tar xvzf FCGI-Client-0.08.tar.gz
```

Puis on utilise dh-make-perl qui va générer les fichiers debian/control, debian/rules &#8230; :

```
dh-make-perl FCGI-Client-0.08/
```

dh-make-perl va indiquer les dépendences à installer avant de compiler le paquet, dans mon cas :

```
= perl >= 5.8.1 is in core
+ Any::Moose >= 0.13 found in libany-moose-perl (>= 0.13)
+ IO::Socket::UNIX  found in perl-base
```

Je les installe donc :

```
apt-get install perl-modules libany-moose-perl perl-base
```

Ensuite, il reste à se placer dans le répertoire et lancer la compilation :

```
cd FCGI-Client-0.08
debuild
```

On devrait se retrouver avec un paquet debian qu&rsquo;il suffira d&rsquo;installer :

```
cd ..
dpkg -i libfcgi-client-perl_0.08-1_all.deb
```
