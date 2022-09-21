---
title: Installer Dell Powervault MD Storage Software (SMcli …) sous Debian
author: VaLouille
type: post
date: 2014-08-27T16:23:15+00:00
url: /2014/08/installer-dell-powervault-md-storage-software-smcli/
categories:
  - linux
  - sysadmin
tags:
  - debian
  - dell
  - md
  - md3000
  - package
  - packages
  - powervault
  - smcli
  - smclient

---
Voici la procédure pour installer SMclient sous Debian en passant par des paquets .DEB :

Java doit être installé :

```
apt-get install openjdk-7-jre
```

Télécharger le ressource CD/DVD sur le site de Dell (Actuellement : DELL\_MDSS\_Consolidated\_RDVD\_5\_1\_0_9.iso)
  
Le monter :

```
mount DELL_MDSS_Consolidated_RDVD_5_1_0_9.iso /mnt
```

On copie les fichiers qui nous intéressent :

```
mkdir ~/dell_mdss
cp /mnt/linux/mdsm/* ~/dell_mdss
cd ~/dell_mdss
```

On crée le lien necessaire pour l&rsquo;exécution du binaire :

```
ln -s /lib/x86_64-linux-gnu/libc.so.6 /lib/libc.so.6
```

On lance le binaire :

```
bash SMIA-LINUXX64.bin -i console
```

Il faut accepter la licence, et laisser les choix par défaut (Typical Full bundle dans /opt/dell/mdstoragemanager)

Ensuite, on va dans le répertoire /opt/dell/mdstoragemanager qui contient les .RPM

```
cd /opt/dell/mdstoragemanager
```

On va avoir besoin d&rsquo;alien pour convertir les .RPM en .DEB

```
apt-get install alien
```

Par exemple, pour créer un paquet Debian pour SMclient (qui contient SMcli), on procède de la manière suivante :

```
alien -g SMclient.rpm --scripts
cd SMclient-11.10.0G06.0020
```

On ajoute la ligne suivante au fichier debian/preinst au dessus du test Java (ligne 9):

```
ln -s /usr/lib/jvm/default-java/jre $RPM_INSTALL_PREFIX/jre
```

On change les lignes suivantes :

```
ln -sf /etc/init.d/SMmonitor /etc/rc.d/rc${i}.d/S99SMmonitor
ln -sf /etc/init.d/SMmonitor /etc/rc.d/rc${i}.d/K00SMmonitor
```

En remplissant comme ceci :

```
ln -sf /etc/init.d/SMmonitor /etc/rc${i}.d/S99SMmonitor
ln -sf /etc/init.d/SMmonitor /etc/rc${i}.d/K00SMmonitor
```

Dans le fichier debian/postinst, on ajoute la ligne ci-dessous au dessus de « /etc/init.d/SMmonitor start »

```
echo "BASEDIR=$RPM_INSTALL_PREFIX" > /var/opt/SM/LAUNCHER_ENV
```

Puis on construit le paquet :

```
dpkg-buildpackage -b
```

Il n&rsquo;y a plus qu&rsquo;à l&rsquo;installer :

```
dpkg -i ../smclient_11.10.0G06.0020-2_all.deb
```
