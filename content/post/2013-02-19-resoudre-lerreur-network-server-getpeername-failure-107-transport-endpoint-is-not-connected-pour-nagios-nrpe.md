---
title: 'Résoudre l&rsquo;erreur Network server getpeername() failure (107: Transport endpoint is not connected) pour Nagios NRPE'
author: VaLouille
type: post
date: 2013-02-19T12:27:58+00:00
url: /2013/02/resoudre-lerreur-network-server-getpeername-failure-107-transport-endpoint-is-not-connected-pour-nagios-nrpe/
categories:
  - sysadmin
tags:
  - keepalived
  - nagios
  - nrpe

---
J&rsquo;ai été confronté à l&rsquo;erreur suivante dans le fichier daemon.log. Ce message s&rsquo;affichait toutes les trois secondes.

```
nrpe[8498]: Error: Network server getpeername() failure (107: Transport endpoint is not connected)
nrpe[8498]: Cannot remove pidfile '/var/run/nagios/nrpe.pid' - check your privileges.
nrpe[8498]: Daemon shutdown
```

C&rsquo;est dû à un bug qui a été reporté <a href="http://comments.gmane.org/gmane.network.nagios.devel/6774" target="_blank">en 2009</a> et qui n&rsquo;est <a href="https://bugs.launchpad.net/ubuntu/+source/nagios-nrpe/+bug/1126890" target="_blank">toujours pas résolu</a> dans la version 2.14.

La commande suivante permet de reproduire le bug :

```
nmap -sT -p 5666 127.0.0.1
```

NRPE va alors supprimer son fichier son fichier nrpe.pid, logguer l&rsquo;erreur, mais continuer à fonctionner. A priori, cela est dû au fait que nmap établit une connexion « half-opened »

Seulement, je ne lance pas un nmap toutes les trois secondes sur la machine. Du coup, à l&rsquo;aide d&rsquo;un tcpdump, on peut voir quelle machine est à l&rsquo;origine de ces connexions, en recoupant avec les heures dans les fichiers daemon.log

```
tcpdump tcp port 5666 and dst 127.0.0.1
```

```
11:55:29.263541 IP 192.168.1.1.20629 > localhost.nrpe: Flags [S], seq 3412324578, win 5840, options [mss 1460,sackOK,TS val 1319828104 ecr 0,nop,wscale 9], length 0 [...]
11:55:35.264239 IP 192.168.1.1.21073 > localhost.nrpe: Flags [S], seq 3081381167, win 5840, options [mss 1460,sackOK,TS val 1319829604 ecr 0,nop,wscale 9], length 0 [...]
11:55:41.265149 IP 192.168.1.1.21521 > localhost.nrpe: Flags [S], seq 3260345897, win 5840, options [mss 1460,sackOK,TS val 1319831104 ecr 0,nop,wscale 9], length 0 [...]
```

```
tail -f /var/log/daemon.log
```

```
Feb 19 11:55:29 localhost nrpe[30875]: Error: Network server getpeername() failure (107: Transport endpoint is not connected)
Feb 19 11:55:29 localhost nrpe[30875]: Cannot remove pidfile '/var/run/nagios/nrpe.pid' - check your privileges.
Feb 19 11:55:29 localhost nrpe[30875]: Daemon shutdown
Feb 19 11:55:35 localhost nrpe[30875]: Error: Network server getpeername() failure (107 [...]
Feb 19 11:55:41 localhost nrpe[30875]: Error: Network server getpeername() failure (107 [...]
```

On voit que l&rsquo;heure est la même. C&rsquo;est donc dans ce cas la machine 192.168.1.1 qui est à l&rsquo;origine de ces requêtes. Il s&rsquo;avère que c&rsquo;est l&rsquo;IP du LVS. En effet, le health checker du LVS lance une requête toutes les 3 secondes afin de vérifier la présence du service. Dans notre cas, avec keepalived, pour desactiver ce « health checker », il suffit de commenter des lignes avec des « !! » dans le fichier /etc/keepalived/keepalived.conf

```
!!       TCP_CHECK &#123;
!!            connect_timeout 10
!!        &#125;
```

Il faut aussi s&rsquo;assurer que cette modification soit faite sur le LVS slave.
