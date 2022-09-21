---
title: "Changer l'adresse ou le domaine de destination dans Postfix"
author: VaLouille
type: post
date: 2014-05-14T15:57:33+00:00
url: /2014/05/changer-ladresse-ou-le-domaine-de-destination-dans-postfix/
categories:
  - linux
  - mail
  - postfix
  - sysadmin
tags:
  - linux
  - postfix
  - rewrite
  - smtp_generic_maps
  - trivial

---
Pour modifier l&rsquo;adresse de destination des mails envoyés dans Postfix, il faut passer par trivial-rewrite. Pour ce faire, il faut ajouter dans le fichier /etc/postfix/main.cf :

```
smtp_generic_maps = regexp:/etc/postfix/generic
```

Ensuite, dans le fichier /etc/postfix/generic

```
/^.*@domain1.com/ user1@domain1.com # renvoyer le domaine vers un utilisateur spécifique
/^.*@domain1.com/ $1@domain2.com    # renvoyer les mails de l'utilisateur de domain1.com vers domain2.com
/^.*@.*/ user1@domain1.com          # tout renvoyer vers user1@domain1.com
```

Pour activer la configuration :

```
postmap /etc/postfix/generic
/etc/init.d/postfix restart
```
