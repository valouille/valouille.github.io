---
title: Créer un dépôt Debian signé « The Debian Way ! »
author: VaLouille
type: post
date: 2013-07-29T15:57:53+00:00
url: /2013/07/creer-un-depot-debian-signe-the-debian-way/
categories:
  - linux
  - sysadmin
tags:
  - debian
  - depot
  - repo

---
Voici la procédure permettant de créer un dépôt Debian signé avec des paquets signés.

Il faut tout d&rsquo;abord générer une clé GPG :

```
apt:~# gpg --gen-key
gpg (GnuPG) 1.4.12; Copyright (C) 2012 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Sélectionnez le type de clef désiré :
   (1) RSA et RSA (par défaut)
   (2) DSA et Elgamal
   (3) DSA (signature seule)
   (4) RSA (signature seule)
Quel est votre choix ? 1
les clefs RSA peuvent faire entre 1024 et 4096 bits de longueur.
Quelle taille de clef désirez-vous ? (2048)
La taille demandée est 2048 bits
Veuillez indiquer le temps pendant lequel cette clef devrait être valable.
         0 = la clef n'expire pas
      <n>  = la clef expire dans n jours
      <n>w = la clef expire dans n semaines
      <n>m = la clef expire dans n mois
      <n>y = la clef expire dans n ans
Pendant combien de temps la clef est-elle valable ? (0)
La clef n'expire pas du tout
Est-ce correct ? (o/N) o

Une identité est nécessaire à la clef ; le programme la construit à partir
du nom réel, d'un commentaire et d'une adresse électronique de cette façon :
   « Heinrich Heine (le poète) <heinrichh@duesseldorf.de> »

Nom réel : Prénom Nom
Adresse électronique : mail@domain.com
Commentaire :
Vous avez sélectionné cette identité :
    « Prénom Nom <mail@domain.com> »

Faut-il modifier le (N)om, le (C)ommentaire, l'(A)dresse électronique
ou (O)ui/(Q)uitter ? O
Une phrase de passe est nécessaire pour protéger votre clef secrète.

De nombreux octets aléatoires doivent être générés. Vous devriez faire
autre chose (taper au clavier, déplacer la souris, utiliser les disques)
pendant la génération de nombres premiers ; cela donne au générateur de
nombres aléatoires une meilleure chance d'obtenir suffisamment d'entropie.

Il n'y a pas suffisamment d'octets aléatoires disponibles. Veuillez faire
autre chose pour que le système d'exploitation puisse rassembler plus
d'entropie (284 octets supplémentaires sont nécessaires).
.+++++
.+++++
De nombreux octets aléatoires doivent être générés. Vous devriez faire
autre chose (taper au clavier, déplacer la souris, utiliser les disques)
pendant la génération de nombres premiers ; cela donne au générateur de
nombres aléatoires une meilleure chance d'obtenir suffisamment d'entropie.
.+++++
.....+++++
gpg: /root/.gnupg/trustdb.gpg : base de confiance créée
gpg: clef FB03BB12 marquée de confiance ultime.
les clefs publique et secrète ont été créées et signées.

gpg: vérification de la base de confiance
-----BEGIN PGP PUBLIC KEY BLOCK-----
gpg: 3 marginale(s) nécessaire(s), 1 complète(s) nécessaire(s),
     modèle de confiance PGP
gpg: profondeur : 0  valables :   1  signées :   0
     confiance : 0 i., 0 n.d., 0 j., 0 m., 0 t., 1 u.
pub   2058R/FB04EE02 2013-07-29
 Empreinte de la clef = B0E1 5914 DD93 44BD 8134  2C28 BD19 BDE1 AD03 EE02
uid                  Prénom Nom <mail@domain.com>
sub   2058R/78443608 2013-07-29
```

La commande suivante va afficher la clé publique. Il faut la copier dans un fichier, par exemple /var/www/depot.key :

```
gpg --armor --export mail@domain.com --output
```

La commande pour la copier :

```
gpg --armor --export mail@domain.com --output > /var/www/depot.key
```

On va avoir besoin de l&rsquo;outil suivant pour signer les paquets

```
apt-get install dpkg-sig
```

Puis on signe le .deb que l&rsquo;on souhaite ajouter au dépot :

```
dpkg-sig --sign builder zabbix-agent_2.0.6-1+awh_amd64.deb
```

L&rsquo;option builder permet de dire que l&rsquo;on est le créateur du paquet

Il va falloir créer un Vhost qui pointe dans /var/www :

```
apt-get install nginx
```

Modifier la ligne root du fichier /etc/nginx/sites-enabled/default comme ceci :

```
root /var/www;
```

Puis on redémarre nginx

```
/etc/init.d/nginx restart
```

Il faut créer les répertoires :

```
mkdir -p /var/www/debian/conf
```

Ensuite, on crée un fichier /var/www/debian/conf/distributions contenant les lignes suivantes (à adapter) :

```
Origin: apt.domain.com
Label: apt repository
Codename: lenny
Architectures: amd64 source
Components: main
Description: debian package repo
SignWith: yes
Pull: lenny
DebIndices: Packages Release . .gz .bz2
UDebIndices: Packages . .gz .bz2
DscIndices: Sources Release .gz .bz2
Contents: . .gz .bz2
```

On utilise reprepro pour gérer nos paquets :

```
apt-get install reprepro
```

Ensuite on se place dans le répertoire /var/www/debian/, puis on ajoute le paquet au dépôt :

```
cd /var/www/debian/
reprepro --ask-passphrase -Vb . includedeb lenny /root/zabbix-agent_2.0.6-1+awh_amd64.deb
```

Et voilà, il ne reste plus qu&rsquo;à ajouter le dépot sur une machine client, ajouter la clé et tester :

```
echo "deb http://apt.domain.com/debian lenny main" >> /etc/apt/sources.list
wget -O - http://apt.domain.com/depot.key | apt-key add -
apt-get update
```

On devrait voir notre dépot :

```
client# apt-cache policy zabbix-agent
 Table de version :
 *** 1:2.0.6-1+awh 0
        500 http://apt.nexen.net lenny/main Packages
        100 /var/lib/dpkg/status
```
