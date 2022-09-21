---
title: Voir quelles IP génèrent ou ont généré le plus de connexions
author: VaLouille
type: post
date: 2013-02-13T14:49:42+00:00
url: /2013/02/voir-quelles-ip-generent-ou-ont-genere-le-plus-de-connexions/
categories:
  - sysadmin
tags:
  - linux
  - netstat

---
Pour savoir si on est en train de subir une attaque, il suffit de passer par netstat, et de présenter les résultats de manière lisible :

```
netstat -laptune | awk '{print $5}' | cut -d":" -f1 | sort | uniq -c | sort -g | tail
```

Le résultat montre les 10 IP qui initialisent le plus de connexions, ainsi que le nombre de connexions.

```
15 192.253.141.64
15 77.224.46.170
148 87.33.146.34
1117 92.130.205.122
1249 92.144.165.55
1279 94.143.232.98
1890 95.102.180.148
2058 95.103.48.12
2129 212.226.73.42
3133 80.223.9.157
```

Pour parser des logs apache ou nginx par exemple, voici comment on peut faire pour présenter les résultats de la même manière :

```
cat /var/log/nginx/access.log |awk '{print $1}' |sort |uniq -c |sort -n | tail
```

```
567 82.120.4.191
698 90.38.39.129
702 92.115.245.78
849 78.214.132.20
865 92.128.64.201
1060 89.117.40.11
1298 89.145.18.20
1420 46.21.224.33
1597 78.87.158.102
1630 92.113.0.173
```
