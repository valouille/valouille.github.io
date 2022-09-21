---
title: Utiliser rsync + SSH avec un port différent de 22
author: VaLouille
type: post
date: 2013-12-19T10:10:50+00:00
url: /2013/12/utiliser-rsync-ssh-avec-un-port-different-de-22/
categories:
  - linux
  - sysadmin
tags:
  - port
  - rsync
  - ssh

---
Pour utiliser rsync + SSH vers un autre port que le port par défaut, on utilise la commande suivante :

```
rsync -av -e 'ssh -p <PORT>' root@IP.IP.IP.IP:/dossier/source /dossier/destination
```

Par exemple avec le port 2222 :

```
rsync -av -e 'ssh -p 2222' root@IP.IP.IP.IP:/dossier/source /dossier/destination
```
