---
title: Faire fonctionner NetApp OnCommand System Manager avec des versions récentes de Java
author: VaLouille
type: post
date: 2015-01-28T10:37:44+00:00
url: /2015/01/faire-fonctionner-netapp-oncommand-system-manager-avec-des-versions-recentes-de-java/
categories:
  - netapp
tags:
  - 7u75
  - 8u31
  - java
  - manager
  - netapp
  - oncommand
  - ssl
  - sslv3
  - system
  - tls

---
Dans les dernières versions de Java, à savoir Java 7u75 et Java 8u31, SSLv3 a été desactivé. Cela peut poser problème avec NetApp OnCommand System Manager sur les systèmes 7-mode. Voici les différents message d&rsquo;erreur qui peuvent en résulter :

Dans OnCommand System Manager :

```
error 500 "connection refused"
```

Dans les logs du NetApp :

```
HTTP XML Authentication failed from IP
```

Le message suivant dans OnCommand System Manager :

```
Set up a secure connection. (Recommended) &nbsp;This option configures SSL using an insecure connection, then switches to secure connection.

Continue without secure connection. &nbsp;Transports credentials and data in clear text format.
```

Dans les logs de OnCommand System Manager :

```
javax.net.ssl.SSLException: Connection has been shutdown: javax.net.ssl.SSLHandshakeException: Server chose SSLv3, but that protocol version is not enabled or not supported by the client.
```

Pour contourner ce problème, il y a deux solutions. La première, la meilleure, étant d&rsquo;activer TLS sur les NetApp sur lesquels ont souhaite se connecter (sur les 2 nodes) :

```
options tls.enable on
```

Il faut ensuite installer la version 3.1.2 RC1 de OnCommand System Manager, car la version 3.1.1 ne supporte pas TLS.

On peut aussi préférer réactiver SSLv3 (non-recommandé), en commentant la dernière ligne dans le fichier C:\Program Files\Java\jre7\lib\security\java.security :

```
# jdk.tls.disabledAlgorithms=SSLv3
```

Un redémarrage de Java est nécessaire afin d&rsquo;appliquer la modification.

Si cela ne fonctionne pas sur les NetApp 7-mode, il est nécessaire de re-générer le certificat :

```
secureadmin setup -f -q ssl t <pays> <departement> <ville> <organisation> <equipe> <fqdn> <email> 1024
```
