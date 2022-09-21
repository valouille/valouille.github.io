---
title: Activer vacation et le faire fonctionner avec roundcube 0.9 et postfixadmin
author: VaLouille
type: post
date: 2014-02-19T11:17:21+00:00
url: /2014/02/activer-vacation-et-le-faire-fonctionner-avec-roundcube-0-9-et-postfixadmin/
categories:
  - linux
  - mail
  - postfix
  - roundcube
  - sysadmin
tags:
  - debian
  - postfixadmin
  - roundcube
  - vacation

---
Voici la procédure pour activer le plugin vacation et le faire fonctionner avec les nouvelles versions de roundcube 0.8.*. domain.com est à changer par un domain qui existe vraiment sur la machine. Ce n&rsquo;est pas la même procédure qu&rsquo;avant, car avec les nouvelles versions de roundcube, les anciens plugins ne fonctionnent plus. Celui-ci a été modifié et traduit.

```
groupadd --gid 10002 vacation
useradd -o -d /var/spool/vacation -g 10002 -u 10002 -s /bin/false vacation
```

Ensuite, créer un dossier pour vacation

```
mkdir /var/spool/vacation
chown -R vacation:vacation /var/spool/vacation
chmod -R 700 /var/spool/vacation
```

Copier le script [vacation.pl][1] (fourni par postfixadmin)

```
cp /usr/share/doc/postfixadmin/examples/VIRTUAL_VACATION/vacation.pl /var/spool/vacation/
```

On installe les dépendances

```
apt-get install libmail-sender-perl libmail-sendmail-perl libmailtools-perl libmime-encwords-perl libemail-valid-perl liblog-log4perl-perl
```

Modifier les informations de base de données du script perl :

```
vim /var/spool/vacation/vacation.pl
```

```
our $db_type = 'mysql';
our $db_host = 'localhost';
our $db_username = 'postfixadmin';
our $db_name = 'postfixadmin';
our $vacation_domain = 'autoreply.domain.com';
our $db_password = 'MOTDEPASSE'; ...
```

Créer le domaine virtuel dans postfix :

```
vim /etc/postfix/transport
```

```
autoreply.domain.com vacation:
```

On regénère le fichier transport.db

```
postmap /etc/postfix/transport
```

Modifier le fichier master.cf

```
vim /etc/postfix/master.cf
```

ajouter en bas :

```
vacation unix - n n - - pipe
flags=DRhu user=vacation argv=/var/spool/vacation/vacation.pl -f$sender ${recipient} ${original_recipient}
```

Ajouter dans /etc/postfix/main.cf

```
transport_maps = hash:/etc/postfix/transport
vacation_destination_recipient_limit = 1
```

On redémarre postfix :

```
/etc/init.d/postfix restart
```

Renseigner le domaine virtuel dans postfixadmin :

```
vim /etc/postfixadmin/config.inc.php
```

```
$CONF['vacation_domain'] = 'autoreply.domain.com';
```

Extraire [pfadmin_autoresponder.tar.gz][2] dans /var/lib/roundcube/plugins

Activer le plugin dans roundcube

```
vim /etc/roundcube/main.inc.php
```

```
$rcmail_config['plugins'] = array('pfadmin_autoresponder');
```

Modifier les infos BDD et domaine du plugin.

```
vim /var/lib/roundcube/plugins/pfadmin_autoresponder/config/config.inc.php
```

Voilà. Il ne reste plus qu&rsquo;à tester en se rendant dans les préférences de roundcube, et checker dans le mail.log qu&rsquo;il n&rsquo;y a pas d&rsquo;erreur.

[<img class="alignnone size-full wp-image-14" alt="roundcube_autoresponse" src="https://blog.valouille.fr/wp-content/uploads/2013/02/roundcube_autoresponse.png" width="700" height="406" srcset="https://blog.valouille.fr/wp-content/uploads/2013/02/roundcube_autoresponse.png 700w, https://blog.valouille.fr/wp-content/uploads/2013/02/roundcube_autoresponse-300x174.png 300w" sizes="(max-width: 700px) 100vw, 700px" />][3]

 [1]: https://blog.valouille.fr/wp-content/uploads/2013/02/vacation.pl.tar.gz
 [2]: https://blog.valouille.fr/wp-content/uploads/2014/02/pfadmin_autoresponder.tar.gz
 [3]: https://blog.valouille.fr/wp-content/uploads/2013/02/roundcube_autoresponse.png
