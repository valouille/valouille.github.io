---
title: Installer Java 6, Java 7 ou Java 8 de SUN/Oracle sous Debian
author: VaLouille
type: post
date: 2013-04-24T16:55:53+00:00
url: /2013/04/installer-java-6-java-7-ou-java-8-de-sunoracle-sous-debian/
categories:
  - Uncategorized
tags:
  - debian
  - java6
  - java7
  - java8

---
Voici la procédure pour installer Java Runtime Environment (jre) sous Debian :
	  

# Installer Java 6
  
Ajouter les sources non-free au fichier /etc/apt/sources.list :

```
deb http://ftp.fr.debian.org/debian/ squeeze main contrib non-free
```

Puis il suffit de mettre à jour la liste des paquets et d&rsquo;installer java :

```
apt-get update
apt-get install sun-java6-plugin sun-java6-jre
```


# Installer Java 7
  
Il faut installer le paquet java-package qui permet de convertir en .deb le tar.gz :

```
apt-get install java-package
```

Puis on télécharge le paquet de Java 7 sur le [site d&rsquo;oracle][1] ou ci-dessous :

```
wget http://download.oracle.com/otn-pub/java/jdk/7u55-b13/jre-7u55-linux-x64.tar.gz
```

On converti le paquet en .deb :

```
make-jpkg jre-7u55-linux-x64.tar.gz
```

Et enfin on installe le paquet :

```
dpkg -i oracle-j2re1.7_1.7.0+update55_amd64.deb
```


# Installer Java 8
  
Il faut installer les outils de compilation de paquets :

```
apt-get install dpkg-dev build-essential git debhelper unzip libasound2 unixodbc libxtst6 curl
```

Puis on clone ce dépot GIT :

```
git clone git://github.com/rraptorr/oracle-java8.git
cd oracle-java8
```

On lance le téléchargement :

```
sh ./prepare.sh
```

On lance la compilation (en root) :

```
dpkg-buildpackage -us -uc
```

Puis on installe les paquets :

```
cd ..
dpkg -i oracle-java8-bin_8.0~b84-1_amd64.deb oracle-java8-jdk_8.0~b84-1_amd64.deb oracle-java8-jre_8.0~b84-1_all.deb oracle-java8-plugin_8.0~b84-1_amd64.deb
```


  * Si on a plusieurs version de java installées, on peut choisir celle que l&rsquo;on veut utiliser par défaut :

```
update-java-alternatives -s java-8-oracle
```

 [1]: http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html
