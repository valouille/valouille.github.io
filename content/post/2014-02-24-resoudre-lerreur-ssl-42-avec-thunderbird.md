---
title: "Résoudre l'erreur SSL 42 avec thunderbird"
author: VaLouille
type: post
date: 2014-02-24T11:02:30+00:00
url: /2014/02/resoudre-lerreur-ssl-42-avec-thunderbird/
categories:
  - linux
  - sysadmin
tags:
  - dovecot
  - imaps
  - ssl
  - thunderbird

---
Quand on essaie d&rsquo;utiliser Thunderbird pour se connecter sur un Dovecot en IMAPS, il se peut que l&rsquo;on se retrouver avec l&rsquo;erreur ci-dessous dans le syslog sur serveur web si le certificat est auto-signé ou invalide :

```
dovecot: imap-login: Disconnected (no auth attempts in 1 secs): user=<>, rip=192.168.0.01, lip=10.10.10.62, TLS: SSL_read() failed: error:14094412:SSL routines:SSL3_READ_BYTES:sslv3 a
lert bad certificate: SSL alert number 42, session=<W2RIsxSzFADAqAM9>
```

Cette erreur est générée car Thunderbird n&rsquo;affiche pas la fenêtre permettant d&rsquo;accepter un certificat non valide pendant la phase d&rsquo;auto-configuration d&rsquo;une adresse de courriel.

Pour contourner ce problème, il suffit de configurer l&rsquo;adresse en IMAP dans l&rsquo;outil de configuration automatique, puis d&rsquo;aller éditer la configuration une fois l&rsquo;adresse créé afin de passer en SSL. Là, Thunderbird va signifier que le certificat est invalide et demander si on souhaite l&rsquo;accepter.
