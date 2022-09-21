---
title: "Récupérer et écrire dans les logs d'accès l'IP source des connexions sur AWS derrière un ELB/ALB avec le header X-Forwarded-For"
date: 2018-03-15T00:05:08+01:00
draft: false
author: VaLouille
type: post
categories:
  - AWS
tags:
  - aws
  - varnish
  - nginx
  - X-Forwarded-for
  - ELB
  - HTTPS
  - ALB
---

Pour récupérer l'adresse IP source du client qui initie la connexion au niveau d'une instance située derrière un Load-Balancer, plutôt que l'IP du Load-Balancer lui-même, il est nécessaire d'utiliser un header HTTP spécifique.

Dans AWS, l'ELB/ALB ajoute [plusieurs headers][1] par défaut, dont [`X-Forwarded-For`][2], qui correspond à l'adresse IP publique à partir de laquelle la connexion est effectuée.

> Les headers ne sont ajoutés que lorsqu'on utilise un Load-Balancer qui fait du niveau 7, c'est à dire un Load-Balancer configuré pour se connecter sur les instances en HTTP et HTTPS, pas en TCP ou SSL (niveau 4). Si ces headers ne sont pas ajoutés par le Load-Balancer, vérifiez la configuration des "Listeners". On peut aussi activer les logs d'accès directement au [niveau de l'ELB/ALB][4].

## Varnish

Pour configurer Varnish NCSA pour qu'il loggue l'IP contenue dans `X-Forwarded-For`, il faut modifier le fichier `/etc/systemd/system/varnishncsa.service` et remplacer le `%%h` par `\"%%{X-Forwarded-For}i\"` :

```
ExecStart=/usr/bin/varnishncsa -a -w /var/log/varnish/access.log -F "\"%%{X-Forwarded-For}i\" %%l %%u %%t \"%%m %%U %%H\" %%s %%b \"%%{Referer}i\" \"%%{User-agent}i\" TimeTaken %%D"
```

Il faut ensuite effectuer un `systemctl daemon-reload` pour prendre en compte la configuration, et redémarrer varnishncsa : `service varnishncsa restart`

On peut visualiser en direct les connexions qui arrivent via la commande `varnishlog` et ainsi vérifier que les IPs sont biens les bonnes :

```
~ # varnishlog | grep X-Forwarded-For
-   ReqHeader      X-Forwarded-For: 1.2.3.4
-   ReqHeader      X-Forwarded-For: 1.2.3.4
-   ReqHeader      X-Forwarded-For: 2.4.6.8
```

## Nginx

### Nginx en tant que serveur web

Pour que Nginx loggue les IP contenues dans `X-Forwarded-For`, il est nécessaire de créer ou modifier un `log_format` dans le fichier `/etc/nginx/nginx.conf` en remplaçant `$remote_addr` par `$http_x_forwarded_for` (ici le `log_format` est nommé `time` ):

```
log_format time '$http_x_forwarded_for - $remote_user [$time_local] [$server_name ] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" [$request_time]';
```

Ensuite, dans le vhost, on configure les logs d'accès :

```
access_log /var/log/nginx/vhost_valouille.log time;
```

On redémarre enfin nginx : `service nginx restart`.

### Nginx en tant que reverse-proxy

Pour passer le header `X-Forwarded-For` à l'application derrière un reverse-proxy Nginx, on met en place les options suivantes dans le vhost :

```
location / {
    proxy_pass http://127.0.0.1:8080;

    proxy_pass_request_headers              on;
    proxy_set_header        Host            $host;
    proxy_set_header        X-Real-IP       $remote_addr;
    proxy_set_header        X-Forwarded-For $http_x_forwarded_for;
}
```

On redémarre enfin nginx : `service nginx restart`.

## Apache

La configuration suivante tirée de [cette documentation][3] permet, si le header `X-Forwarded-For` est présent et contient une IP, d'utiliser le `LogFormat` adéquat :

```
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%{X-Forwarded-For}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" proxy
SetEnvIf X-Forwarded-For "^.*\..*\..*\..*" forwarded
CustomLog "logs/access_log" combined env=!forwarded
CustomLog "logs/access_log" proxy env=forwarded
```

En mode reverse-proxy, avec le module `remoteip` activé (`a2enmod remoteip`), il suffit d'ajouter la ligne `RemoteIPHeader X-Forwarded-For` dans la définition du backend pour passer le header à l'application sous-jacente.

[1]: https://docs.aws.amazon.com/fr_fr/elasticloadbalancing/latest/classic/x-forwarded-headers.html
[2]: https://docs.aws.amazon.com/fr_fr/elasticloadbalancing/latest/classic/x-forwarded-headers.html#x-forwarded-for
[3]: http://www.loadbalancer.org/blog/apache-and-x-forwarded-for-headers/
[4]: https://docs.aws.amazon.com/fr_fr/elasticloadbalancing/latest/classic/access-log-collection.html
