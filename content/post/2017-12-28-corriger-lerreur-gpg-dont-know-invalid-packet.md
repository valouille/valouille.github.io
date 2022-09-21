---
title: "Corriger l'erreur gpg: [don't know]: invalid packet"
author: VaLouille
type: post
date: 2017-12-28T16:40:07+00:00
url: /2017/12/corriger-lerreur-gpg-dont-know-invalid-packet/
categories:
  - linux
  - sysadmin

---
J&rsquo;ai été confronté à l&rsquo;erreur suivante en essayant d&rsquo;ajouter une clé GPG à APT :

```
~ # apt-key add keyring.gpg
gpg: [don't know]: invalid packet (ctb=2d)
gpg: keyring_get_keyblock: read error: Invalid packet
gpg: keydb_get_keyblock failed: Invalid keyring
gpg: [don't know]: invalid packet (ctb=2d)
gpg: /tmp/apt-key-gpghome.kpq2RJAPsu/pubring.gpg: copy to '/tmp/apt-key-gpghome.kpq2RJAPsu/pubring.gpg.tmp' failed: Invalid packet
gpg: error writing keyring '/tmp/apt-key-gpghome.kpq2RJAPsu/pubring.gpg': Invalid packet
gpg: [don't know]: invalid packet (ctb=2d)
gpg: keyring_get_keyblock: read error: Invalid packet
gpg: error reading keyblock: Invalid keyring
gpg: error reading 'keyring.gpg': Invalid packet
gpg: import from 'keyring.gpg' failed: Invalid packet
```

Cette erreur se manifeste lorsqu&rsquo;une clé GPG est endommagée. Bien qu&rsquo;il soit théoriquement possible via la commande « apt-key list » d&rsquo;identifier la clé qui pose problème, et de restaurer le fichier qui se termine par « ~ », je ne suis pas parvenu à corriger le problème de cette manière.

Le seul moyen que j&rsquo;ai trouvé pour corriger cette erreur a été de supprimer toutes les clés et de réinstaller les paquets qui mettent en place les clés pour les dépôts Debian (en spécifiant les options qui rétablissent les fichiers de configuration) :

```
rm /etc/apt/trusted.gpg
rm /etc/apt/trusted.gpg.d/*
apt-get -o DPkg::options::=--force-confmiss --reinstall install debian-keyring debian-archive-keyring
```

On peut utiliser les commandes suivantes pour réinstaller les clés des dépôts tiers :

```
gpg --keyserver pgpkeys.mit.edu --recv-key 1285436434D8686F
gpg -a --export 1285436434D8686F | apt-key add -
```
