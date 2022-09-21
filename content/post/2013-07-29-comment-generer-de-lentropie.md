---
title: "Comment générer de l'entropie"
author: VaLouille
type: post
date: 2013-07-29T13:24:38+00:00
url: /2013/07/comment-generer-de-lentropie/
categories:
  - linux
  - sysadmin
tags:
  - debian
  - entropie
  - gpg

---
Quand j&rsquo;ai essayé de créer une clé GPG sur une machine fraichement installé, j&rsquo;ai été confronté au message suivant :

```
De nombreux octets aléatoires doivent être générés. Vous devriez faire
autre chose (taper au clavier, déplacer la souris, utiliser les disques)
pendant la génération de nombres premiers ; cela donne au générateur de
nombres aléatoires une meilleure chance d'obtenir suffisamment d'entropie.
```

Afin de voir l&rsquo;entropie disponible, on peut utiliser la commande suivante :

```
watch cat /proc/sys/kernel/random/entropy_avail
```

Pour en générer, on peut utiliser la commande suivante :

```
apt-get install -y rng-tools
/usr/sbin/rngd -r /dev/urandom
```
