---
title: Supprimer le fichier hiberfil.sys dans Windows Vista/7/8
author: VaLouille
type: post
date: 2014-08-14T09:27:17+00:00
url: /2014/08/supprimer-le-fichier-hiberfil-sys-dans-windows-vista78/
categories:
  - windows
tags:
  - 7
  - 8
  - 8.1
  - hiberfil.sys
  - seven
  - vista
  - windows

---
Windows Vista, Seven et 8 utilisent un fichier C:\hiberfil.sys

Ce fichier peut être assez volumineux car il est utilisé pour stocker les informations contenues dans la RAM lors d&rsquo;une mise en veille prolongée. Donc si on a 8G de RAM, il peut faire jusqu&rsquo;à 8GO. C&rsquo;est le principe du « Suspend to Disk »

Afin de le supprimer, il faut désactiver la mise en veille prolongée. Pour cela, ouvrez l&rsquo;invite de commande (Exécuter en tant qu&rsquo;administrateur), et tapez la commande suivante :

```
powercfg -h off
```

Le fichier sera supprimé instantanément.

L&rsquo;utilisation de la mise en veille « normale » (Suspend to RAM) sera toujours possible
