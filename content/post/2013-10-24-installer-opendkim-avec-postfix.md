---
title: Installer OpenDKIM avec Postfix
author: VaLouille
type: post
date: 2013-10-24T12:42:05+00:00
url: /2013/10/installer-opendkim-avec-postfix/
categories:
  - linux
  - sysadmin
tags:
  - debian
  - dkim
  - mail
  - opendkim
  - postfix

---
DKIM permet d&rsquo;authentifier le nom de domaine de l&rsquo;expéditeur d&rsquo;un mail. Cela peut être utile pour ajouter de la fiabilité à ses mails et ainsi éviter qu&rsquo;ils se trouvent classés comme « spam ».

DKIM s&rsquo;installe à côté et se couple à Postfix. Voici la procédure pour installer opendkim, et ainsi authentifier les mails qui seront envoyés, à supposer que postfix est déjà fonctionnel.

On installe opendkim

```
apt-get install opendkim
```

On met en place la configuration dans le fichier /etc/opendkim.conf en changeant le domaine

```
Domain                  domaine.fr
KeyFile                 /etc/postfix/opendkim/private.key
Selector                dkim
```

On crée le répertoire qui contiendra les clés publiques/privées, et on les génére :

```
mkdir /etc/postfix/opendkim
cd /etc/postfix/opendkim
openssl genrsa -out private.key 1024
openssl rsa -in private.key -pubout -out public.key
chmod 640 private.key
chown opendkim private.key
```

On active opendkim en anjoutant la ligne suivante dans le fichier /etc/default/opendkim

```
SOCKET="inet:8891:localhost"
```

Puis on modifie la configuration de postfix (/etc/postfix/main.cf) pour ajouter les lignes suivantes à la fin

```
milter_default_action = accept
milter_protocol = 6
smtpd_milters = inet:127.0.0.1:8891
non_smtpd_milters = inet:127.0.0.1:8891
```

Il faut ajouter dans le configuration DNS du domaine les entrées suivantes:

```
_domainkey IN TXT "t=y; o=-;"
dkim._domainkey IN TXT "k=rsa; t=s; p=cle_publique"
```

En remplaçant `clé_publique` par le contenu du fichier /etc/postfix/opendkim/public.key

Puis on redémarre les services

```
/etc/init.d/postfix restart && /etc/init.d/opendkim restart ; tail -f /var/log/mail.*
```

Pour vérifier le bon fonctionnement, il faut envoyer un mail, puis regarder dans les headers qu&rsquo;on a bien la ligne suivante :

```
DKIM-Signature: v=1; a=rsa-sha256; c=simple/simple; d=domaine.fr; s=dkim;
    t=1355221579; bh=XhK90OInN6V6XilH51qAngENZwcNmUlEyhqaYYz6qfA=;
    h=To:Subject:Message-Id:Date:From;
    b=Zpthor6xHLcoDUkzpuaUVeN0k/v2RPSrd6X70mY8OCey0dDiDHyjuCr68BGzJqI62
     38clqy0HzyGDyIafOAjGtdXTa+G+wnbfyPc4JyuzdQBffPk3XMtDZ3A4WQcxyhOR/2
     vP+nuHBhnMQlby62FLDkSRh+RkxJAv8RuzKEXjM8=
```

Avec bind, il faut éventuellement ajouter les lignes suivantes à named.conf

```
check-names master ignore;
check-names slave  ignore;
```
