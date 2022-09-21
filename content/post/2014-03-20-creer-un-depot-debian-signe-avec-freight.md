---
title: Créer un repository Debian signé avec Freight
author: VaLouille
type: post
date: 2014-03-20T10:40:20+00:00
url: /2014/03/creer-un-depot-debian-signe-avec-freight/
categories:
  - linux
  - sysadmin
tags:
  - debian
  - depot
  - freight
  - repo
  - repository

---
Reprepro ne permet pas d&rsquo;avoir plusieurs versions d&rsquo;un même paquet dans son dépôt. Pour contourner cette limitation, je suis donc passé à <a href="https://github.com/rcrowley/freight" title="Freight" target="_blank">Freight</a>.

Freight : 

  * permet d&rsquo;ajouter des .deb directement sans fichier .changes
  * signe à la volée les paquets
  * autorise plusieurs versions d&rsquo;un même paquet
  * permet d&rsquo;avoir plusieurs branches
  * est très simple à installer et à maintenir
  * est très léger et n&rsquo;a pas besoin de base SQL


L&rsquo;installation est ultra simple :

```
echo "deb http://packages.rcrowley.org $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/rcrowley.list
wget -O /etc/apt/trusted.gpg.d/rcrowley.gpg http://packages.rcrowley.org/keyring.gpg
apt-get update
apt-get -y install freight
```

On génère ensuite une clé GPG :

```
gpg --gen-key
```

```
gpg (GnuPG) 1.4.12; Copyright (C) 2012 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 2048
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: Internal Repository
Email address: repository@exemple.com
Comment:
You selected this USER-ID:
    "Internal Repository <repository@exemple.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.
```

On peut entrer une passphrase ou pas.

Ensuite, on initialise le dépôt (ici je crée 2 branches) :

```
mkdir /var/www/freight
cd /var/www/freight
freight-init --gpg=repo@exemple.com --libdir=/var/www/freight/lib --cachedir=/var/www/freight/cache --archs="i386 amd64 all" --origin="My Internal Repository" --label="My Internal Reposiroty" --suite="wheezy-php53 wheezy-php54"
```

Puis pour ajouter des paquets, la commande est la suivante :

```
freight add php5-cli_5.3.26_amd64.deb php5-cli_5.3.27_amd64.deb apt/wheezy-php53
freight cache apt/wheezy-php53
```

Voilà, il ne reste plus qu&rsquo;à créer un Vhost qui point vers /var/www/freight/cache

```
Alias /debian   /space/www/freight/cache
<Directory "/space/www/freight/cache">
      Options Indexes FollowSymLinks MultiViews
      AllowOverride Options
      Order allow,deny
      allow from all
</Directory>
```

Les lignes à ajouter dans le fichier /etc/apt/sources.list des machines clientes seront de la forme suivante :

```
deb http://exemple.com/debian wheezy-php54 main
deb http://exemple.com/debian wheezy-php53 main
```

Pour ajouter la clé publique sur les machines clientes :

```
wget -O /etc/apt/trusted.gpg.d/exemple.gpg http://exemple.com/debian/keyring.gpg
```

Et voilà :

```
# apt-cache policy php5-cli
php5-cli:
  Installed: (none)
  Candidate: 5.5.9+dfsg-1
  Version table:
     5.5.9+dfsg-1 0
        500 http://ftp.fr.debian.org/debian/ unstable/main amd64 Packages
     5.3.27-1 0
        500 http://exemple.com/debian/ wheezy-php53/main amd64 Packages
     5.3.26-1 0
        500 http://exemple.com/debian/ wheezy-php53/main amd64 Packages
```

Si la clé a une passphrase et qu&rsquo;on ne souhaite pas la taper à chaque ajout de paquet, on peut créer un fichier contenant la passphrase, et ajouter l&rsquo;option suivante dans le fichier de configuration de Freight :

```
GPG_PASSPHRASE_FILE="/root/.passphrase"
```
