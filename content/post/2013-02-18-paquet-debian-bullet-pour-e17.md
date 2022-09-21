---
title: Paquet Debian Bullet pour compiler e17
author: VaLouille
type: post
date: 2013-02-18T11:05:05+00:00
url: /2013/02/paquet-debian-bullet-pour-e17/
categories:
  - linux
tags:
  - debian e17 bullet ephysics enlightenment

---
Lors de la compilation de enlightenment, j&rsquo;ai eu l&rsquo;erreur suivante :

```
configure: Start EPhysics checks
configure: error: pkg-config missing bullet >= 2.80
```

[Bullet][1] est une librairie physique.

Bullet n&rsquo;étant pas disponible sous Debian, je l&rsquo;ai compilé. Pour l&rsquo;installer, il faut le télécharger ici : [bullet-2.81\_2.81+rev2613-1\_amd64.deb.tar.gz][2]

```
tar xvzf bullet-2.81_2.81+rev2613-1_amd64.deb.tar.gz
dpkg -i bullet-2.81_2.81+rev2613-1_amd64.deb
```

Pour l&rsquo;installer à la main :

```
apt-get install cmake freeglut3-dev
wget https://bullet.googlecode.com/files/bullet-2.81-rev2613.tgz
tar xvzf bullet-2.81-rev2613.tgz
cd bullet-2.81-rev2613
cmake -D BUILD_SHARED_LIBS=true .
make
make install
```

On vérifie que c&rsquo;est bien installé :

```
ldconfig -v | grep -i bullet
ldconfig: Path `/usr/lib/x86_64-linux-gnu' given more than once
    libBulletDynamics.so.2.81 -> libBulletDynamics.so.2.81
    libBulletCollision.so.2.81 -> libBulletCollision.so.2.81
    libBulletSoftBody.so.2.81 -> libBulletSoftBody.so.2.81
```

 [1]: https://code.google.com/p/bullet/ "Bullet"
 [2]: https://blog.valouille.fr/wp-content/uploads/2013/02/bullet-2.81_2.81+rev2613-1_amd64.deb.tar.gz
