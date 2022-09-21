---
title: "Réinstaller les fichiers de configuration par défaut d'un paquet debian"
author: VaLouille
type: post
date: 2013-08-09T09:49:06+00:00
url: /2013/08/reinstaller-les-fichiers-de-configuration-par-defaut-dun-paquet-debian/
categories:
  - linux
  - sysadmin
tags:
  - configuration
  - debian
  - défaut
  - fichiers
  - paquet
  - reinstaller

---
Si on a modifié des fichiers de configuration d&rsquo;un paquet debian et que l&rsquo;on souhaite remettre les fichiers de configuration originaux, voici la démarche à suivre :

On vérifie que le(s) fichier(s) de configuration sont bien dans le paquet</li> 

```
dpkg --status <paquet>
```

Exemple avec puppet :

```
Conffiles:
 /etc/init.d/puppet 79dec4169533326b9e1a21aac681d8e8
 /etc/default/puppet 9e5a0cf174ccff1af10342297b8b1bdb
```

Si je veux remettre le fichier /etc/init.d/puppet par défaut, il faut que je le renomme :

```
mv /etc/init.d/puppet /etc/init.d/puppet.broken
```

Ensuite, je le réinstalle

  * Avec apt-get :

```
apt-get -o DPkg::options::=--force-confmiss --reinstall install <package>
```

  * Avec dpkg :

```
dpkg -i --force-confmiss <paquet.deb>
```
