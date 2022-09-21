---
title: Supprimer les mails reçus en double avec Pigeonhole et Dovecot 2
author: VaLouille
type: post
date: 2013-02-18T15:15:39+00:00
url: /2013/02/supprimer-les-mails-recus-en-double-avec-pigeonhole-et-dovecot-2/
categories:
  - linux
  - mail
  - sysadmin
tags:
  - dovecot
  - duplicate
  - pigeonhole
  - sieve

---
Si on est inscrit à deux mailing-lists, et qu&rsquo;un mail est envoyé à ces deux mailing-lists, alors on va le recevoir deux fois. Ou même si on est simplement en copie du mail envoyé à la mailing list. Il est possible de supprimer ces doublons, en se basant sur [vnd.dovecot.duplicate][1]. C&rsquo;est une extension sieve. Le principe est que chaque mail contient un « msgid » unique, et le plugin se base sur cet ID afin de détecter les doublons.

Sous Debian, Pingeonhole s&rsquo;installe de la façon suivante :

```
apt-get install dovecot-sieve dovecot-managesieved
```

Ensuite, il faut activer le plugin dans la configuration dovecot. Le plugin est disponible à partir de la version 0.3.1 de Pigeonhole.

```
vim /etc/dovecot/conf.d/90-sieve.conf
```

```
plugin {
  sieve = ~/.dovecot.sieve
  sieve_default = /var/lib/dovecot/sieve/default.sieve
  sieve_dir = ~/sieve
  sieve_global_dir = /var/lib/dovecot/sieve/global/
  sieve_extensions = +vnd.dovecot.duplicate
}
```

Il faut redémarrer dovecot pour que les changements soient pris en compte:

```
/etc/init.d/dovecot restart
```

Puis on crée le répertoire dans lequel on va placer notre règle globale (pour tous les utilisateurs) :

```
mkdir -p /var/lib/dovecot/sieve/global
chown -R vmail:mail /var/lib/dovecot
chmod u+x /var/lib/dovecot
```

Il est aussi possible d&rsquo;appliquer cette règle uniquement pour certains utilisateurs, voir la <a href="http://wiki2.dovecot.org/Pigeonhole/Sieve/Configuration" title="Documentation Dovecot" target="_blank">documentation Dovecot</a>
  
Enfin, on crée la règle dans le fichier /var/lib/dovecot/sieve/default.sieve

```
require ["vnd.dovecot.duplicate", "fileinto", "mailbox", "imap4flags"]; 

if header :contains "X-Spam-Status" "Yes" { 
        fileinto "Spam"; 
} 

if duplicate {
        setflag "\\seen";
        fileinto "Trash"; 
}
```

On place les droits :

```
chown vmail:mail /var/lib/dovecot/sieve/default.sieve
```

Cette règle place le spam dans un répertoire spam, et place les doublons dans la corbeille, en les marquant comme &lsquo;lus&rsquo;
  
Il ne reste plus qu&rsquo;à tester. Par défaut, les erreurs sont logguées dans le fichier /var/log/mail.log.

 [1]: http://hg.rename-it.nl/dovecot-2.1-pigeonhole/raw-file/tip/doc/rfc/spec-bosch-sieve-duplicate.txt "vnd.dovecot.duplicate"
