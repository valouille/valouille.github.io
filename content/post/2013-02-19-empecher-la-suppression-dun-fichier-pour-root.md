---
title: "Empêcher la suppression/modification d'un fichier par root"
author: VaLouille
type: post
date: 2013-02-19T15:09:38+00:00
url: /2013/02/empecher-la-suppression-dun-fichier-pour-root/
categories:
  - linux
  - sysadmin
tags:
  - chattr
  - linux
  - root

---
Pour empêcher la modification ou la suppression d&rsquo;un fichier, y compris pour root, on peut utiliser la commande « chattr » qui permet de mettre en place des attributs avancés sur les fichiers.

  * Pour bloquer un fichier :
```
chattr +i fichier
```

  * Pour voir les attributs avancés d&rsquo;un fichier :
```
lsattr
----i----------- ./fichier
```

  * Pour débloquer un fichier
```
chattr -i fichier
```
