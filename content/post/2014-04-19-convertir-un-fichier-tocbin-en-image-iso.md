---
title: Convertir un fichier toc/bin en image ISO
author: VaLouille
type: post
date: 2014-04-19T22:15:56+00:00
url: /2014/04/convertir-un-fichier-tocbin-en-image-iso/
categories:
  - linux
tags:
  - bin
  - cue
  - iso
  - toc

---
Brasero (via cdrdao) génère des images disque sous forme de fichiers .toc/.bin.

Pour les convertir, voici la procédure :

* Sous Linux :

```
apt-get install cdrdao bchunk
```

* Sous Mac :

```
sudo port install cdrdao bchunk
```
    
Ensuite il faut convertir le fichier .toc en fichier .cue :
   
``` 
toc2cue fichier.toc fichier.cue
```
   
Puis on peut à partir de ces deux fichiers générer une image ISO :

```    
bchunk fichier.bin fichier.cue fichier.iso
```
