---
title: Concaténer deux fichiers, ou les merger une ligne sur deux
author: VaLouille
type: post
date: 2013-02-19T15:26:19+00:00
url: /2013/02/concatener-deux-fichiers-ou-les-merger-une-ligne-sur-deux/
categories:
  - linux
tags:
  - bash
  - linux

---
Parfois, on a une liste dans les mains sous forme de fichier texte, et on doit insérer du contenu à la fin, ou une ligne sur deux. Pour cela, on peut utiliser la commande « paste ».

Prennons le cas de ces deux fichiers :

```
% cat fichier1.txt
111
222
333
```

```
% cat fichier2.txt 
aaa
bbb
ccc
```

Si on veut la concaténer, voici la commande :

```
% paste -d ' ' fichier1.txt fichier2.txt
111 aaa
222 bbb
333 ccc
```

Et si on souhaite insérer en dessous de chaque ligne, on remplace le délimiteur par un retour chariot :

```
% paste -d '\n' fichier1.txt fichier2.txt 
111
aaa
222
bbb
333
ccc
```

Pour sauvegarder le résultat dans un fichier, on redirige la sortie standard :

```
% paste -d '\n' fichier1.txt fichier2.txt > fichier3.txt
```

Selon les besoins, on peut utiliser n&rsquo;importe quel délimiteur, tel qu&rsquo;un point virgule ou un slash par exemple
