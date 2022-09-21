---
title: Mettre en place une limite sur le nombre de mails envoyés par IP sur Postfix
author: VaLouille
type: post
date: 2014-09-04T09:57:02+00:00
url: /2014/09/mettre-en-place-une-limite-sur-le-nombre-de-mails-envoyes-par-ip-sur-postfix/
categories:
  - linux
  - mail
  - postfix
  - sysadmin
tags:
  - limit
  - mail
  - postfix
  - rate
  - throttle

---
Dans Postfix, on a la possibilité de paramétrer une limite du nombre de mails envoyés sur une intervalle de temps. Par exemple, pour mettre en place une limite de 10 mails envoyés par minutes, on ajoute les lignes suivantes au fichier `/etc/postfix/main.cf` :

```
# Whitelist
smtpd_client_event_limit_exceptions = 127.0.0.0/8
# Max 10 mails/IP/anvil_rate_time_unit
smtpd_client_message_rate_limit = 10
# Reject pipelining (speed-up mail sending)
smtpd_data_restrictions = reject_unauth_pipelining
smtpd_delay_reject = yes
# Reset counter each minute
anvil_rate_time_unit = 60s
# Log statistics each 5 minutes
anvil_status_update_time = 600s
```

Pour activer les modifications :

```
/etc/init.d/postfix reload
```

Chaque fois qu&rsquo;un client atteint cette limite, il reçoit l&rsquo;erreur suivante et met le mail en deferred :

```
-Queue ID- --Size-- ----Arrival Time---- -Sender/Recipient-------
0DD51F82        274 Tue Sep  2 15:52:28  postmaster@domain.com
(host 192.168.3.14[192.168.4.14] said: 450 4.7.1 Error: too much mail from 192.168.3.12 (in reply to MAIL FROM command))
                                         email@domain.com
```

L&rsquo;erreur est logguée de cette façon dans les logs de postfix :

```
Sep  2 16:04:07 machine.local postfix/smtpd[25359]: warning: Message delivery request rate limit exceeded: 55 from domain1.com[192.168.3.12] for service smtp
```

Afin d&rsquo;avoir la liste des machines qui ont atteint la limite sur la journée en cours, on peut utiliser la commande suivante (peut prendre jusqu&rsquo;à 2 minutes) :

```
/bin/cat /var/log/mail.log | grep "Message delivery request rate limit exceeded" |  grep "`date +%b' '%e`" | while read ligne ; do echo $ligne | awk '{print $(NF-5),"\t",$(NF-3)}' ; done | sort -k1 -g | uniq > /tmp/liste_rate_limit && for i in `/bin/cat /tmp/liste_rate_limit | cut -d'[' -f2 | cut -d ']' -f1 | sort | uniq`; do /bin/cat /tmp/liste_rate_limit | grep "$i" | sort -k1 -g | tail -1 ; done | sort -k1 -gr
```
