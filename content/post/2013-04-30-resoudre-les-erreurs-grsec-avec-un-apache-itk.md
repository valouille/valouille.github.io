---
title: Résoudre les erreurs Grsec avec un Apache ITK
author: VaLouille
type: post
date: 2013-04-30T15:21:26+00:00
url: /2013/04/resoudre-les-erreurs-grsec-avec-un-apache-itk/
categories:
  - linux
  - sysadmin
tags:
  - apache
  - grsec
  - kernel

---
Grsec utilise par défaut les GID 1002, 1003, 1004 et 1005. Cela peut empêcher certaines applications de fonctionner, par exemple un Apache ITK configuré avec des GID utilisés par grsec. Les erreurs logguées dans le kernel ressemblent à ça :

```
kernel: grsec: From IP.IP.IP.IP: denied connect() by /usr/sbin/apache2[apache2:21077] uid/euid:1003/1003 gid/egid:1003/1003, parent /usr/sbin/apache2[apache2:21072] uid/euid:0/0 gid/egid:0/0
```

Il y a deux solutions, soit on désactive les fonctions qui nous embêtent :

```
sysctl kernel.grsecurity.socket_all=
sysctl kernel.grsecurity.tpe=
sysctl kernel.grsecurity.socket_client=
sysctl kernel.grsecurity.socket_server=
```

Soit on change les GID utilisés :

```
sysctl socket_all_gid=10004
sysctl socket_client_gid=10003
sysctl socket_server_gid=10002
sysctl tpe_gid=10005
```

> Il ne faut pas oublier de redémarrer Apache, et d&rsquo;ajouter ces lignes (avec un espace avant et après le &lsquo;=&rsquo;) dans /etc/sysctl.conf
  
Pour information, voici l&rsquo;utilité de ces paramètres :

  * socket_all : Définit un groupe d&rsquo;utilisateurs non autorisés à se connecter à d&rsquo;autres hôtes ni à lancer d&rsquo;applications serveur.
  * socket\_all\_gid : 1004 par défaut.
  * socket_client : Définit un groupe d&rsquo;utilisateurs non autorisés à se connecter à d&rsquo;autres hôtes mais elle pourront lancer des applications serveur.
  * socket\_client\_gid : 1003 par défaut
  * socket_server : Définit un groupe d&rsquo;utilisateurs non autorisés à lancer des applications serveurs.
  * socket\_server\_gid : 1002 par défaut
  * tpe : Définit un groupe d&rsquo;utilisateurs non dignes de confiance qui ne pourront pas exécuter des commandes non contenues dans un répertoire appartenant à root et dont l&rsquo;écriture est réservée à root.
