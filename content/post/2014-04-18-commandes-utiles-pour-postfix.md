---
title: Commandes utiles pour Postfix
author: VaLouille
type: post
date: 2014-04-18T15:21:10+00:00
url: /2014/04/commandes-utiles-pour-postfix/
categories:
- linux
- mail
- postfix
- sysadmin
tags:
- mail
- postfix
- queue

---
### Voir quels domaines ont beaucoup de mails deferred en réception : <div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:830px;">
```
qshape deferred
```

### Voir quels domaines ont beaucoup de mails actifs en émission : <div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:830px;">
```
qshape -s active
```

### Supprimer tous les mails dans « Deferred » <div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:830px;">
```
postsuper -d ALL deferred
```

### Supprimer tous les mails en queue <div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:830px;">
```
postsuper -d ALL
```

### Supprimer un mail spécifique <div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:830px;">
```
postsuper -d DA80E24A0A
```

### Voir le contenu d&rsquo;un mail dans la queue <div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:830px;">
```
postcat -q F1F942D236
```

### Supprimer les mails de la queue en fonction de l’expéditeur ou du destinataire <div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:830px;">
```
mailq | tail -n+2 | awk 'BEGIN { RS = "" } { if ($8 == "www-data@exemple.com" && $9 == "")print $1 }' | tr -d '*!' | postsuper -d -
```

On peut choisir :

$7 &#8211; Sender

$8 &#8211; Recipient

$9 &#8211; Recipient2

### Flush la queue : <div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:830px;">
```
postfix flush
```

### Voir la queue : <div class="codecolorer-container bash default" style="overflow:auto;white-space:nowrap;width:830px;">
```
mailq
```

### Whitelister/Blacklister des adresses/IP/domaines/ranges : 

Ajouter au fichier /etc/postfix/main.cf :

```
smtpd_recipient_restrictions = check_sender_access hash:/etc/postfix/sender_access, check_client_access hash:/etc/postfix/client_checks
```

```
/etc/postfix/client_checks
# ceux qui ne peuvent pas envoyer des mails vers une adresse gérée sur ce serveur

example.com               REJECT Ce domaine envoie du SPAM
.example.com              REJECT Ce sous-domaine envoie du SPAM
11.11.22.22               REJECT Cette IP envoie du SPAM
33.33.44.0/24             REJECT Ce range envoie du SPAM
55.55.66.66               OK
example2.com              OK
```

```
/etc/postfix/sender_checks
# ceux qui peuvent envoyer des mails à partir du serveur

example.com              REJECT Domaine non autorise
.example.com             REJECT Sous-domaine non autorise
utilisateur@example.com  REJECT Cet utilisateur n'est pas autorise
example2.com             OK
```

Pour appliquer :

```
postmap hash:sender_access<br /> postmap hash:client_access<br /> /etc/init.d/postfix reload
```
