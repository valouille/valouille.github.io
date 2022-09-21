---
title: Utiliser rsnapshot avec des processus rsync en parallèle
author: VaLouille
type: post
date: 2013-02-13T18:22:21+00:00
url: /2013/02/utiliser-rsnapshot-avec-des-processus-rsync-en-parallele/
categories:
  - sysadmin
tags:
  - backup
  - rsnapshot

---
rsnapshot est un outil qui permet d&rsquo;automatiser les backups en utilisant la combinaison « rsync/cp -al ». Seulement, il lit son fichier de configuration de manière séquentielle et donc backup les machines une à une. Si les machines sources sont bridées en sortie, ou que on utilise du bonding, cette limitation peut être ennuyeuse. Pour passer outre, il faut procéder comme ceci :

Il faut déjà créer un script bash. Dans ce cas, on lance 2 rsync en parallèle :

```
#!/bin/bash 

groups="01 02" 

# Run backup jobs in parallel 
for group in ${groups}; do 
    rsnapshot -q -c /etc/rsnapshot/group${group}.conf sync  &
done 

# Wait for all backup jobs to finish 
wait 

# Rotate daily backup 
rsnapshot -q -c /etc/rsnapshot/rotate.conf daily
```

Ensuite, la configuration se fait de la façon suivante :
  
/etc/rsnapshot/rsnapshot.conf

```
config_version  1.2
snapshot_root   /var/backup/rsnapshot/
cmd_cp      /bin/cp
cmd_rm      /bin/rm
cmd_rsync   /usr/bin/rsync
cmd_logger  /usr/bin/logger
retain      daily   3
verbose     2
loglevel    3
lockfile    /var/run/rsnapshot.pid
sync_first  1
```

un fichier /etc/rsnapshot/rotate.conf :

```
include_conf    /etc/rsnapshot/rsnapshot.conf
logfile /var/log/rsnapshot-rotate.log
lockfile    /var/run/rsnapshot-rotate.pid
include_conf    /etc/rsnapshot/group01.hosts
include_conf    /etc/rsnapshot/group02.hosts
```

un fichier /etc/rsnapshot/group01.conf

```
include_conf    /etc/rsnapshot/rsnapshot.conf
logfile     /var/log/rsnapshot1.log
lockfile    /var/run/rsnapshot1.pid
include_conf    /etc/rsnapshot/group01.hosts
```

Un fichier /etc/rsnapshot/group01.hosts

```
backup  rsync://example.com/backup  example.com/
backup  rsync://example2.conf/backup    example2.com/
[...]
```

Il faut faire pareil pour les fichiers group02.conf et group02.hosts, en modifiant logfile, lockfile et include_conf. Il faut bien sûr ne pas renseigner les mêmes machines dans group01.hosts et group02.hosts
  
Il suffit ensuit de lancer le script créé au dessus.

Le fonctionnement est le suivant :
  
[<img src="https://blog.valouille.fr/wp-content/uploads/2013/02/rsnapshot.png" alt="rsnapshot" width="638" height="413" class="alignnone size-full wp-image-90" srcset="https://blog.valouille.fr/wp-content/uploads/2013/02/rsnapshot.png 638w, https://blog.valouille.fr/wp-content/uploads/2013/02/rsnapshot-463x300.png 463w" sizes="(max-width: 638px) 100vw, 638px" />][1]

 [1]: https://blog.valouille.fr/wp-content/uploads/2013/02/rsnapshot.png
