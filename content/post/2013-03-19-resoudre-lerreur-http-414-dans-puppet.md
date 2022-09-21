---
title: "Résoudre l'erreur HTTP 414 dans puppet"
author: VaLouille
type: post
date: 2013-03-19T16:36:34+00:00
url: /2013/03/resoudre-lerreur-http-414-dans-puppet/
categories:
  - linux
  - sysadmin
tags:
  - 414
  - error
  - http
  - passenger
  - puppet

---
Avec la version 2.6 de puppet utilisée en Squeeze, on peut rencontrer l&rsquo;erreur suivante :

```
err: Could not retrieve catalog from remote server: Error 414 on SERVER: <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>414 Request-URI Too Large</title>
</head><body>
<h1>Request-URI Too Large</h1>
<p>The requested URL's length exceeds the capacity
limit for this server.<br />
```

Ce bug est résolu dans la version 2.7 de puppet. 

On peut soit mettre à jour le client via les backports :

```
apt-get install -t squeeze-backports puppet
```

Soit modifier la configuration du serveur comme ceci :

```
vim /usr/lib/ruby/1.9.1/webrick/httprequest.rb
```

Augmenter la valeur suivante ;

```
MAX_URI_LENGTH = 4096
```

Si on utilise puppet avec Passenger, il faut ajouter dans le VHost Apache l&rsquo;option suivante

```
<VirtualHost *:8140>
    [...]
    LimitRequestLine  15000
```

Puis on restart Apache.
