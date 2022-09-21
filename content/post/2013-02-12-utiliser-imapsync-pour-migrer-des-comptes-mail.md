---
title: Utiliser imapsync pour migrer des comptes mail
author: VaLouille
type: post
date: 2013-02-12T14:19:15+00:00
url: /2013/02/utiliser-imapsync-pour-migrer-des-comptes-mail/
categories:
  - mail
  - sysadmin
tags:
  - imapsync
  - mail

---
[Imapsync][1] permet de synchroniser les mails d&rsquo;un compte imap à l&rsquo;autre. Il fonctionne de manière incrémentale et ne crée pas de doublons. On peut donc le lancer plusieurs fois. On peut soit :

  * l&rsquo;installer via git :
```
git clone https://github.com/imapsync/imapsync.git
apt-get install libmail-imapclient-perl libdigest-md5-file-perl libterm-readkey-perl libio-socket-ssl-perl libfile-spec-perl libdigest-hmac-perl
make
make install
```

  * soit par paquet Debian : [imapsync\_1.518+awh-1\_amd64.deb.tar.gz][2]
```
tar xvzf imapsync_1.518+awh-1_amd64.deb.tar.gz
dpkg -i imapsync_1.518+awh-1_amd64.deb
apt-get -f install
```

Le fonctionnement est le suivant :

```
imapsync --host1 IP_IMAP_SOURCE --user1 "USER_SOURCE" --password1 "PASSWORD_SOURCE" --host2 IP_IMAP_DESTINATION --user2 "USER_DESTINATION" --password2 "PASSWORD_DESTINATION"
```

Si on veut automatiser, pour migrer plusieurs utilisateurs d&rsquo;un coup, il faut d&rsquo;abord créer un fichier, par exemple imapsync_users.txt :

```
user_01_1;password_01_1;user_01_2;password_01_2
user_02_1;password_02_1;user_02_2;password_02_2
[...]
```

Puis utiliser le script ci-dessous, en modifiant l&rsquo;IP source et destination :

```
#!/bin/bash
{ while IFS=';' read  u1 p1 u2 p2
    do 
         { echo "$u1" | egrep "^#" ; } > /dev/null && continue
         NOW=`date +%Y_%m_%d_%H_%M_%S` 
         echo syncing to user "$u2"
         imapsync --host1 IP_IMAP_SOURCE --user1 "$u1" --password1 "$p1" \
                  --host2 IP_IMAP_DESTINATION --user2 "$u2" --password2 "$p2" \
                  > /tmp/imapsync_log_${u2}_$NOW.txt 2>&1
    done 
} < imapsync_users.txt
```

 [1]: https://github.com/imapsync/imapsync "Imapsync"
 [2]: https://blog.valouille.fr/wp-content/uploads/2013/02/imapsync_1.518+awh-1_amd64.deb.tar.gz
