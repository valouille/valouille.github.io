---
title: Corriger l'erreur 106 Illegal character in VCL name (‘:’) avec Varnish 5
author: VaLouille
type: post
date: 2018-01-04T15:31:45+00:00
url: /2018/01/corriger-lerreur-106-illegal-character-in-vcl-name-avec-varnish-5/
categories:
  - linux
  - sysadmin

---
J&rsquo;ai été confronté à l&rsquo;erreur suivante lors du reload du démon varnish :

```
Wr 200 PONG 1515077296 1.0
Rd vcl.load root:1515087296.307831546 /etc/varnish/default.vcl
Wr 106 Illegal character in VCL name (':')
```

Ce problème provient du fichier /usr/share/varnish/reload-vcl qui n&rsquo;est pas écrasé par les version 5.0 ou 5.1 de varnish lorsqu&rsquo;on met à jour. Les lignes suivantes génèrent un fichier vcl avec un nom qui n&rsquo;est plus pris en compte par Varnish :

```
if [ -f /proc/sys/kernel/random/uuid ]
then
uuid=$(cat /proc/sys/kernel/random/uuid)
vcl_label="vcl_${uuid}"
else
vcl_label="vcl_$($date +%s-%N)"
fi
```

Pour corriger le problème, il faut modifier les lignes de la manière suivante :

```
if [ -f /proc/sys/kernel/random/uuid ]
then
uuid=$(cat /proc/sys/kernel/random/uuid)
vcl_label="vcl_${LOGNAME}${LOGNAME:+_}${uuid}"
else
vcl_label="vcl_$($date +${LOGNAME}${LOGNAME:+_}%s-%N)"
fi
```

On peut aussi supprimer le fichier et réinstaller varnish :

```
rm /usr/share/varnish/reload-vcl && apt-get install varnish --reinstall
```
