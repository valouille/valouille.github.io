---
title: "Empêcher les mails d'être délivrés"
author: VaLouille
type: post
date: 2014-03-28T13:36:14+00:00
url: /2014/03/empecher-les-mails-detre-delivres/
categories:
  - linux
  - sysadmin
tags:
  - mail
  - postfix
  - queue

---
Afin d&#8217;empêcher les mails de certains domaines d&rsquo;être délivrés, par exemple en cas de maintenance ou de migration du domaine, on peut laisser ces mails dans la queue en les mettant en status « HOLD » dans postfix.

Pour passer un domaine en HOLD, il faut modifier le fichier /etc/postfix/main.cf tel que ci-dessous et redémarrer postfix :

```
smtpd_recipient_restrictions = 
    ...
    check_recipient_access hash:/etc/postfix/hold
```

Ensuite on ajoute les lignes suivantes dans le fichier /etc/postfix/hold :

```
example1.com        HOLD
example2.com         HOLD
```

Afin d&rsquo;appliquer les changements, on utilise la commande suivante :

```
postmap /etc/postfix/hold
```

Pour libérer les mails en HOLD :

```
postsuper -r ALL
```
